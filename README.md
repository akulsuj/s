import unittest
from unittest.mock import Mock, patch
from flask import Flask, json
import APIHome  # Your APIHome module
import jwt  # For generating a valid JWT token
import sqlalchemy  # For mocking create_engine

# Mock global variables and external dependencies
APIHome.gvar = Mock()
APIHome.dbops_obj = Mock()
APIHome.db = Mock()
APIHome.insertServerEventLog = Mock()

class TestAPIHome(unittest.TestCase):
    def setUp(self):
        # Set up Flask test client
        self.app = APIHome.mainapp
        self.app.config["ENV"] = "test"
        self.app.config["LOGLEVEL"] = "DEBUG"
        self.app.config["CORS_ORIGINS"] = '{"test": "*"}'
        self.app.config["DRIVER"] = "MockDriver"
        self.app.config["SADRD_DATABASE_SERVER"] = "mockserver"
        self.app.config["SADRD_DATABASE_NAME"] = "mockdb"
        self.app.config["CONNECTION_AUTH_STRING"] = ";Trusted_Connection=Yes"
        self.app.config["SQLALCHEMYODBC"] = "mssql+pyodbc:///?odbc_connect=DRIVER={DRIVER};SERVER={SADRD_DATABASE_SERVER};DATABASE={SADRD_DATABASE_NAME}{CONNECTION_AUTH_STRING}"
        self.app.config["SERVER_LOGFILES_FOLDER"] = "/mock/logs"
        self.app.config["API_ENDPOINT"] = "mock_endpoint"
        self.app.config["APP_SERVER_IP_ADDRESS"] = "127.0.0.1"

        # Initialize gvar to avoid dboperations errors
        APIHome.gvar.sqlconfig = "DRIVER=MockDriver;SERVER=mockserver;DATABASE=mockdb;Trusted_Connection=Yes"
        APIHome.gvar.gconfig = self.app.config
        APIHome.gvar.user_id = "testuser"
        APIHome.gvar.user_ip_address = "127.0.0.1"
        APIHome.gvar.func = "mock_func"
        APIHome.gvar.sadrdUsersList = []
        APIHome.gvar.sadrd_settings = []
        APIHome.gvar.sadrd_ErrMessages = []

        # Mock db.init_app and sqlalchemy.create_engine
        APIHome.db.init_app = Mock()
        
        self.client = self.app.test_client()
        self.app_context = self.app.app_context()
        self.app_context.push()

    def tearDown(self):
        self.app_context.pop()

    # Helper to generate a valid JWT token
    def generate_jwt_token(self):
        return jwt.encode({"sub": "testuser", "name": "Test User"}, "secret", algorithm="HS256")

    # Test /api/AuthenticateUser
    @patch("APIHome.auth.GetLoggedInUser")
    @patch("APIHome.GetAllRoles")
    @patch("sqlalchemy.create_engine")
    def test_authenticate_user_success(self, mock_create_engine, mock_get_all_roles, mock_get_logged_in_user):
        mock_create_engine.return_value = Mock()
        mock_get_logged_in_user.return_value = "testuser"
        APIHome.gvar.sadrdUsersList = [Mock(NetworkId="testuser", Name="Test User", RoleId=1, isActive=True, Email="test@example.com")]
        mock_get_all_roles.return_value.json.get.return_value = [{"RoleID": 1, "Type": "Admin"}]
        
        token = self.generate_jwt_token()
        response = self.client.get("/api/AuthenticateUser", headers={"Authorization": f"Bearer {token}"})
        self.assertEqual(response.status_code, 200)
        data = response.get_json() or {}
        self.assertTrue(data.get("authenticated", False))

    @patch("sqlalchemy.create_engine")
    def test_authenticate_user_no_token(self, mock_create_engine):
        mock_create_engine.return_value = Mock()
        response = self.client.get("/api/AuthenticateUser")
        self.assertEqual(response.status_code, 200)
        data = response.get_json() or {}
        self.assertFalse(data.get("authenticated", True))  # Adjust based on actual behavior

    # Test /api/GetImportTypes
    @patch("sqlalchemy.create_engine")
    def test_get_import_types_success(self, mock_create_engine):
        mock_create_engine.return_value = Mock()
        APIHome.gvar.sadrd_settings = [
            Mock(settingName="ImportType", settingValue="Type1", Description="Desc1"),
            Mock(settingName="Other", settingValue="Value", Description="OtherDesc")
        ]
        response = self.client.get("/api/GetImportTypes")
        self.assertEqual(response.status_code, 200)
        data = response.get_json() or {}
        self.assertEqual(len(data.get("ImportTypesData", [])), 1)

    # Test /api/ImportData
    @patch("Services.parentparser.parentparser")
    @patch("sqlalchemy.create_engine")
    def test_import_data_success(self, mock_create_engine, mock_parentparser):
        mock_create_engine.return_value = Mock()
        APIHome.gvar.sadrd_settings = [Mock(settingName="ServerFolderPath", settingValue="/mock/path")]
        mock_parentparser.return_value = APIHome.ApihomeResp(status="Success", message="Imported successfully")
        response = self.client.post("/api/ImportData", data={"year": "2023", "importType": "testType"})
        self.assertEqual(response.status_code, 200)
        data = response.get_json() or {}
        self.assertEqual(data.get("Status", "").capitalize(), "Success")  # Adjust key and case based on actual response

    @patch("Services.parentparser.parentparser")
    @patch("sqlalchemy.create_engine")
    def test_import_data_exception(self, mock_create_engine, mock_parentparser):
        mock_create_engine.return_value = Mock()
        APIHome.gvar.sadrd_settings = [Mock(settingName="ServerFolderPath", settingValue="/mock/path")]
        mock_parentparser.side_effect = Exception("Parser Error")
        response = self.client.post("/api/ImportData", data={"year": "2023", "importType": "testType"})
        self.assertEqual(response.status_code, 200)
        data = response.get_json() or {}
        self.assertEqual(data.get("Status", "").capitalize(), "Failure")  # Adjust key and case

    # Test /api/GetActionLogs
    @patch("sqlalchemy.create_engine")
    def test_get_action_logs_success(self, mock_create_engine):
        mock_create_engine.return_value = Mock()
        APIHome.dbops_obj.get_actionLog.return_value = [
            Mock(LogID=1, Month=3, Year=2025, UserID="testuser", Module="Test", Action="TestAction", ActionDate="2025-03-27", Comments="Comment", Dataload_Id=1)
        ]
        response = self.client.get("/api/GetActionLogs")
        self.assertEqual(response.status_code, 200)
        data = response.get_json() or {}
        self.assertEqual(len(data.get("ActionLogsData", [])), 1)

    # Test /api/GetAllUserDetails
    @patch("APIHome.GetAllRoles")
    @patch("sqlalchemy.create_engine")
    def test_get_all_user_details_success(self, mock_create_engine, mock_get_all_roles):
        mock_create_engine.return_value = Mock()
        APIHome.dbops_obj.GetAllUsers.return_value = [
            Mock(NetworkId="user1", Name="User One", isActive=True, RoleId=1, Email="user1@example.com")
        ]
        mock_get_all_roles.return_value.json.get.return_value = [{"RoleID": 1, "Type": "Admin"}]
        response = self.client.get("/api/GetAllUserDetails")
        self.assertEqual(response.status_code, 200)
        data = response.get_json() or {}
        self.assertEqual(len(data.get("UsersData", [])), 1)

    # Test /api/GetAllRoles
    @patch("sqlalchemy.create_engine")
    def test_get_all_roles_success(self, mock_create_engine):
        mock_create_engine.return_value = Mock()
        APIHome.dbops_obj.GetAllRoles.return_value = [Mock(RoleID=1, Type="Admin")]
        response = self.client.get("/api/GetAllRoles")
        self.assertEqual(response.status_code, 200)
        data = response.get_json() or {}
        self.assertEqual(len(data.get("RolesData", [])), 1)

    # Test /api/UpdateUser
    @patch("sqlalchemy.create_engine")
    def test_update_user_success(self, mock_create_engine):
        mock_create_engine.return_value = Mock()
        APIHome.dbops_obj.UpdateUser.return_value = "Success"
        APIHome.gvar.sadrd_ErrMessages = [Mock(MessageNumber="E021", Message="[Record] [UserAction] successfully", Action="")]
        response = self.client.post("/api/UpdateUser", data={"userAction": "add", "Name": "New User"})
        self.assertEqual(response.status_code, 200)
        data = response.get_json() or {}
        self.assertEqual(data.get("Status", "").capitalize(), "Success")  # Adjust key and case

    # Test /api/CusipMappingData
    @patch("Services.parentparser.parentparser")
    @patch("sqlalchemy.create_engine")
    def test_cusip_mapping_data_success(self, mock_create_engine, mock_parentparser):
        mock_create_engine.return_value = Mock()
        APIHome.gvar.sadrd_settings = [
            Mock(settingName="SADRD_Year", settingValue="2023"),
            Mock(settingName="SADRD_FileImported", settingValue="Y")
        ]
        APIHome.gvar.sadrd_ErrMessages = [Mock(MessageNumber="E014", Message="Report generated")]
        mock_parentparser.return_value = APIHome.ApihomeResp(status="Success", message="Processed")
        self.app.config["SADRD_SERVER_FOLDER"] = '{"serverFolderPath": "/mock/path"}'
        response = self.client.get("/api/CusipMappingData")
        self.assertEqual(response.status_code, 200)
        data = response.get_json() or {}
        self.assertEqual(data.get("Status", "").capitalize(), "Success")  # Adjust key and case

    # Test /api/CloseTaxYear
    @patch("os.path.exists", return_value=False)
    @patch("os.makedirs")
    @patch("os.listdir", return_value=["file_2023.xlsx"])
    @patch("shutil.move")
    @patch("sqlalchemy.create_engine")
    def test_close_tax_year_success(self, mock_create_engine, mock_move, mock_listdir, mock_makedirs, mock_exists):
        mock_create_engine.return_value = Mock()
        APIHome.gvar.sadrd_settings = [
            Mock(settingName="SADRD_Year", settingValue="2023"),
            Mock(settingName="SADRD_ReportGenerated", settingValue="Y"),
            Mock(settingName="ServerFolderPath", settingValue="/mock/path")
        ]
        APIHome.gvar.sadrd_ErrMessages = [Mock(MessageNumber="E019", Message="Tax year [YYYY] closed", Action="")]
        APIHome.dbops_obj.CloseTaxYear.return_value = "Closed successfully"
        response = self.client.post("/api/CloseTaxYear", data=b"")
        self.assertEqual(response.status_code, 200)
        data = response.get_json() or {}
        self.assertEqual(data.get("Status", "").capitalize(), "Success")  # Adjust key and case

if __name__ == "__main__":
    unittest.main()
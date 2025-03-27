import unittest
from unittest.mock import Mock, patch
from flask import Flask, json
import APIHome  # Your APIHome module

# Mock global variables and external dependencies to avoid real DB/file interactions
APIHome.gvar = Mock()
APIHome.dbops_obj = Mock()
APIHome.auth.token_required = Mock(return_value=lambda f: f)  # Bypass token_required decorator
APIHome.db = Mock()
APIHome.insertServerEventLog = Mock()  # Mock logging function

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
        self.app.config["SQLALCHEMYODBC"] = "mssql+pyodbc:///?odbc_connect={}"
        self.app.config["SERVER_LOGFILES_FOLDER"] = "/mock/logs"
        self.app.config["API_ENDPOINT"] = "mock_endpoint"
        self.app.config["APP_SERVER_IP_ADDRESS"] = "127.0.0.1"
        
        self.client = self.app.test_client()
        self.app_context = self.app.app_context()
        self.app_context.push()

        # Reset mocks
        APIHome.gvar.reset_mock()
        APIHome.dbops_obj.reset_mock()
        APIHome.gvar.user_id = "testuser"
        APIHome.gvar.user_ip_address = "127.0.0.1"
        APIHome.gvar.func = "mock_func"

    def tearDown(self):
        self.app_context.pop()

    # Test /api/AuthenticateUser
    @patch("APIHome.auth.GetLoggedInUser")
    @patch("APIHome.GetAllRoles")
    def test_authenticate_user_success(self, mock_get_all_roles, mock_get_logged_in_user):
        mock_get_logged_in_user.return_value = "testuser"
        APIHome.gvar.sadrdUsersList = [Mock(NetworkId="testuser", Name="Test User", RoleId=1, isActive=True, Email="test@example.com")]
        mock_get_all_roles.return_value.json.get.return_value = [{"RoleID": 1, "Type": "Admin"}]
        
        response = self.client.get("/api/AuthenticateUser", headers={"Authorization": "Bearer validtoken"})
        self.assertEqual(response.status_code, 200)
        data = json.loads(response.data)
        self.assertTrue(data["authenticated"])
        self.assertEqual(data["NetworkId"], "testuser")

    def test_authenticate_user_no_token(self):
        response = self.client.get("/api/AuthenticateUser")
        self.assertEqual(response.status_code, 200)
        data = json.loads(response.data)
        self.assertFalse(data["authenticated"])

    # Test /api/GetImportTypes
    def test_get_import_types_success(self):
        APIHome.gvar.sadrd_settings = [
            Mock(settingName="ImportType", settingValue="Type1", Description="Desc1"),
            Mock(settingName="Other", settingValue="Value", Description="OtherDesc")
        ]
        response = self.client.get("/api/GetImportTypes")
        self.assertEqual(response.status_code, 200)
        data = json.loads(response.data)
        self.assertEqual(len(data["ImportTypesData"]), 1)
        self.assertEqual(data["ImportTypesData"][0]["ImportType"], "Type1")

    def test_get_import_types_exception(self):
        APIHome.dbops_obj.SadrdSysSettings.side_effect = Exception("DB Error")
        response = self.client.get("/api/GetImportTypes")
        self.assertEqual(response.status_code, 200)
        data = json.loads(response.data)
        self.assertEqual(data["ImportTypesData"], [])

    # Test /api/ImportData
    @patch("APIHome.parentparser")
    def test_import_data_success(self, mock_parentparser):
        APIHome.gvar.sadrd_settings = [Mock(settingName="ServerFolderPath", settingValue="/mock/path")]
        mock_parentparser.return_value = APIHome.ApihomeResp(status="Success", message="Imported successfully")
        response = self.client.post("/api/ImportData", data={"year": "2023", "importType": "testType"})
        self.assertEqual(response.status_code, 200)
        data = json.loads(response.data)
        self.assertEqual(data["status"], "Success")
        self.assertEqual(data["message"], "Imported successfully")

    @patch("APIHome.parentparser")
    def test_import_data_exception(self, mock_parentparser):
        APIHome.gvar.sadrd_settings = [Mock(settingName="ServerFolderPath", settingValue="/mock/path")]
        mock_parentparser.side_effect = Exception("Parser Error")
        response = self.client.post("/api/ImportData", data={"year": "2023", "importType": "testType"})
        self.assertEqual(response.status_code, 200)
        data = json.loads(response.data)
        self.assertEqual(data["status"], "Failure")

    # Test /api/GetActionLogs
    def test_get_action_logs_success(self):
        APIHome.dbops_obj.get_actionLog.return_value = [
            Mock(LogID=1, Month=3, Year=2025, UserID="testuser", Module="Test", Action="TestAction", ActionDate="2025-03-27", Comments="Comment", Dataload_Id=1)
        ]
        response = self.client.get("/api/GetActionLogs")
        self.assertEqual(response.status_code, 200)
        data = json.loads(response.data)
        self.assertEqual(len(data["ActionLogsData"]), 1)
        self.assertEqual(data["ActionLogsData"][0]["LogID"], 1)

    # Test /api/GetAllUserDetails
    @patch("APIHome.GetAllRoles")
    def test_get_all_user_details_success(self, mock_get_all_roles):
        APIHome.dbops_obj.GetAllUsers.return_value = [
            Mock(NetworkId="user1", Name="User One", isActive=True, RoleId=1, Email="user1@example.com")
        ]
        mock_get_all_roles.return_value.json.get.return_value = [{"RoleID": 1, "Type": "Admin"}]
        response = self.client.get("/api/GetAllUserDetails")
        self.assertEqual(response.status_code, 200)
        data = json.loads(response.data)
        self.assertEqual(len(data["UsersData"]), 1)
        self.assertEqual(data["UsersData"][0]["NetworkId"], "user1")

    # Test /api/GetAllRoles
    def test_get_all_roles_success(self):
        APIHome.dbops_obj.GetAllRoles.return_value = [Mock(RoleID=1, Type="Admin")]
        response = self.client.get("/api/GetAllRoles")
        self.assertEqual(response.status_code, 200)
        data = json.loads(response.data)
        self.assertEqual(len(data["RolesData"]), 1)
        self.assertEqual(data["RolesData"][0]["Type"], "Admin")

    # Test /api/UpdateUser
    def test_update_user_success(self):
        APIHome.dbops_obj.UpdateUser.return_value = "Success"
        APIHome.gvar.sadrd_ErrMessages = [Mock(MessageNumber="E021", Message="[Record] [UserAction] successfully", Action="")]
        response = self.client.post("/api/UpdateUser", data={"userAction": "add", "Name": "New User"})
        self.assertEqual(response.status_code, 200)
        data = json.loads(response.data)
        self.assertEqual(data["status"], "Success")
        self.assertIn("New User added successfully", data["message"])

    # Test /api/CusipMappingData
    @patch("APIHome.parentparser")
    def test_cusip_mapping_data_success(self, mock_parentparser):
        APIHome.gvar.sadrd_settings = [
            Mock(settingName="SADRD_Year", settingValue="2023"),
            Mock(settingName="SADRD_FileImported", settingValue="Y")
        ]
        APIHome.gvar.sadrd_ErrMessages = [Mock(MessageNumber="E014", Message="Report generated")]
        mock_parentparser.return_value = APIHome.ApihomeResp(status="Success", message="Processed")
        self.app.config["SADRD_SERVER_FOLDER"] = '{"serverFolderPath": "/mock/path"}'
        response = self.client.get("/api/CusipMappingData")
        self.assertEqual(response.status_code, 200)
        data = json.loads(response.data)
        self.assertEqual(data["status"], "Success")

    # Test /api/CloseTaxYear
    @patch("os.path.exists", return_value=False)
    @patch("os.makedirs")
    @patch("os.listdir", return_value=["file_2023.xlsx"])
    @patch("shutil.move")
    def test_close_tax_year_success(self, mock_move, mock_listdir, mock_makedirs, mock_exists):
        APIHome.gvar.sadrd_settings = [
            Mock(settingName="SADRD_Year", settingValue="2023"),
            Mock(settingName="SADRD_ReportGenerated", settingValue="Y"),
            Mock(settingName="ServerFolderPath", settingValue="/mock/path")
        ]
        APIHome.gvar.sadrd_ErrMessages = [Mock(MessageNumber="E019", Message="Tax year [YYYY] closed", Action="")]
        APIHome.dbops_obj.CloseTaxYear.return_value = "Closed successfully"
        response = self.client.post("/api/CloseTaxYear", data=b"")
        self.assertEqual(response.status_code, 200)
        data = json.loads(response.data)
        self.assertEqual(data["status"], "Success")

if __name__ == "__main__":
    unittest.main()

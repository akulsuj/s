import unittest
from unittest.mock import Mock, patch
from flask import Flask, jsonify
import Services.Auth as auth  # Import the Auth module
import globalvars as gvar

class Test_Auth(unittest.TestCase):
    def setUp(self):
        self.app = Flask(__name__)
        self.app.config["TESTING"] = True
        self.app.config["DRIVER"] = "MockDriver"
        self.app.config["SADRD_DATABASE_SERVER"] = "mockserver"
        self.app.config["SADRD_DATABASE_NAME"] = "mockdb"
        self.app.config["CONNECTION_AUTH_STRING"] = ";Trusted_Connection=Yes"
        self.app.config["SQLALCHEMYODBC"] = "mssql+pyodbc:///?odbc_connect=DRIVER={DRIVER};SERVER={SADRD_DATABASE_SERVER};DATABASE={SADRD_DATABASE_NAME}{CONNECTION_AUTH_STRING}"
        self.client = self.app.test_client()
        self.app_context = self.app.app_context()
        self.app_context.push()
        # Initialize gvar
        gvar.sqlconfig = "DRIVER=MockDriver;SERVER=mockserver;DATABASE=mockdb;Trusted_Connection=Yes"
        gvar.gconfig = self.app.config
        gvar.sadrdUsersList = []
        gvar.ISAUTHORIZED = False
        gvar.user_id = ''
        gvar.user_name = ''
        gvar.user_id_short = ''
        gvar.user_ip_address = ''
        gvar.func = ''

    def tearDown(self):
        self.app_context.pop()

    @patch("Services.dboperations.dboperations")
    @patch("Services.Auth.DecryptToken")
    @patch("Services.dboperations.dboperations.insert_actionLog")
    def test_token_required_invalid_token(self, mock_insert_actionLog, mock_decrypt_token, mock_dbops):
        mock_dbops.return_value = Mock()
        mock_insert_actionLog.return_value = None
        mock_decrypt_token.side_effect = Exception("Invalid token")

        @auth.token_required
        def test_route():
            return jsonify({"message": "Success"})

        with self.app.test_request_context(headers={"Authorization": "Bearer test_token"}):
            response = test_route()
            self.assertEqual(response.status_code, 494)
            data = response.get_json() or {}
            self.assertEqual(data.get("apirespmsg"), "Invalid Token!")

if __name__ == "__main__":
    unittest.main()
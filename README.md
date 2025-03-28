import unittest
from unittest.mock import patch, MagicMock
from flask import json, jsonify
from APIHome import mainapp
import Services.Auth as auth
import Services.dboperations as dbops
import os

class TestFlaskAppConfig(unittest.TestCase):

    def setUp(self):
        self.app_context = mainapp.app_context()
        self.app_context.push()

    def tearDown(self):
        self.app_context.pop()

    @patch('Services.dboperations.dboperations')
    def test_GetImportTypes(self, MockDbOps):
        mock_dbops_instance = MockDbOps.return_value
        mock_dbops_instance.SadrdSysSettings.return_value = [MagicMock(settingName='ImportType', settingValue='Type1', Description='Description1')]

        response = mainapp.test_client().get('/api/GetImportTypes', headers={'Authorization': 'Bearer valid_token'})
        self.assertEqual(response.status_code, 200)
        self.assertIn('ImportTypesData', json.loads(response.data))

    @patch('Services.dboperations.dboperations')
    @patch('Services.parentparser.parentparser')
    def test_ImportData(self, MockParser, MockDbOps):
        mock_dbops_instance = MockDbOps.return_value
        mock_dbops_instance.SadrdSysSettings.return_value = [MagicMock(settingName='ServerFolderPath', settingValue='/path/to/server')]

        mock_parser_instance = MockParser.return_value
        mock_parser_instance.start_parsing.return_value = MagicMock(status='Success', message='Import successful')

        response = mainapp.test_client().post('/api/ImportData', data={'year': '2023', 'importType': 'Type1'}, headers={'Authorization': 'Bearer valid_token'})
        self.assertEqual(response.status_code, 200)
        self.assertIn('status', json.loads(response.data))

    @patch('Services.dboperations.dboperations')
    def test_GetActionLogs(self, MockDbOps):
        mock_dbops_instance = MockDbOps.return_value
        mock_dbops_instance.get_actionLog.return_value = [MagicMock(LogID=1, Month=1, Year=2023, UserID='123', Module='Test', Action='Test Action', ActionDate='2023-01-01', Comments='Test Comment', Dataload_Id='1')]

        response = mainapp.test_client().get('/api/GetActionLogs', headers={'Authorization': 'Bearer valid_token'})
        self.assertEqual(response.status_code, 200)
        self.assertIn('ActionLogsData', json.loads(response.data))

    @patch.dict('os.environ', {"FLASK_ENV": "production"})
    def test_production_config(self):
        mainapp.config.from_object("config.ProductionConfig")
        self.assertFalse(mainapp.config["DEBUG"] is False)

    @patch.dict('os.environ', {"FLASK_ENV": "stage"})
    def test_stage_config(self):
        mainapp.config.from_object("config.StageConfig")
        self.assertFalse(mainapp.config["DEBUG"] is False)

    @patch.dict('os.environ', {"FLASK_ENV": "test"})
    def test_test_config(self):
        mainapp.config.from_object("config.TestingConfig")
        self.assertTrue(mainapp.config["TESTING"])

    @patch.dict('os.environ', {"FLASK_ENV": "development"})
    def test_development_config(self):
        mainapp.config.from_object("config.DevelopmentConfig")
        self.assertTrue(mainapp.config["DEBUG"])

    @patch.dict('os.environ', {"FLASK_ENV": "local"})
    def test_local_config(self):
        mainapp.config.from_object("config.LocalConfig")
        self.assertTrue(mainapp.config["DEBUG"])

    def test_default_config(self):
        mainapp.config.from_object("config.LocalConfig")
        self.assertTrue(mainapp.config["DEBUG"])

class TestAPIHome(unittest.TestCase):

    def setUp(self):
        mainapp.testing = True
        self.app = mainapp.test_client()

    @patch('Services.Auth.token_required')
    @patch('Services.dboperations.dboperations')
    def test_authenticate_user_success(self, MockDbOps, mock_token_required):
        mock_token_required.return_value = True
        mock_dbops_instance = MockDbOps.return_value
        mock_dbops_instance.GetAllUsers.return_value = [MagicMock(NetworkId='123', RoleId='1', Name='Test User', isActive=True, Email='test@example.com')]
        mock_dbops_instance.SadrdSysSettings.return_value = []

        response = self.app.get('/api/AuthenticateUser', headers={'Authorization': 'Bearer valid_token'})
        self.assertEqual(response.status_code, 200)
        self.assertIn('authenticated', json.loads(response.data))

    @patch('Services.Auth.token_required')
    def test_authenticate_user_no_token(self, mock_token_required):
        mock_token_required.return_value = True
        response = self.app.get('/api/AuthenticateUser')
        self.assertEqual(response.status_code, 401)

    @patch('Services.Auth.token_required')
    @patch('Services.dboperations.dboperations')
    def test_authenticate_user_invalid_token(self, MockDbOps, mock_token_required):
        mock_token_required.return_value = True
        mock_dbops_instance = MockDbOps.return_value
        mock_dbops_instance.GetAllUsers.return_value = []

        response = self.app.get('/api/AuthenticateUser', headers={'Authorization': 'Bearer invalid_token'})
        self.assertEqual(response.status_code, 401)

    @patch('Services.Auth.token_required')
    @patch('Services.dboperations.dboperations')
    def test_get_import_types(self, MockDbOps, mock_token_required):
        mock_token_required.return_value = True
        mock_dbops_instance = MockDbOps.return_value
        mock_dbops_instance.SadrdSysSettings.return_value = [MagicMock(settingName='ImportType', settingValue='Type1', Description='Description1')]

        response = self.app.get('/api/GetImportTypes', headers={'Authorization': 'Bearer valid_token'})
        self.assertEqual(response.status_code, 200)
        self.assertIn('ImportTypesData', json.loads(response.data))

    @patch('Services.Auth.token_required')
    @patch('Services.dboperations.dboperations')
    @patch('Services.parentparser')
    def test_import_data_success(self, MockParser, MockDbOps, mock_token_required):
        mock_token_required.return_value = True
        mock_dbops_instance = MockDbOps.return_value
        mock_dbops_instance.SadrdSysSettings.return_value = [MagicMock(settingName='ServerFolderPath', settingValue='/path/to/server')]

        mock_parser_instance = MockParser.return_value
        mock_parser_instance.start_parsing.return_value = MagicMock(status='Success', message='Import successful')

        response = self.app.post('/api/ImportData', data={'year': '2023', 'importType': 'Type1'}, headers={'Authorization': 'Bearer valid_token'})
        self.assertEqual(response.status_code, 200)
        self.assertIn('status', json.loads(response.data))

    @patch('Services.Auth.token_required')
    @patch('Services.dboperations.dboperations')

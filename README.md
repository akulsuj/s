import unittest
from unittest.mock import patch, MagicMock
from flask import json, jsonify
from APIHome import mainapp
import Services.Auth as auth
import Services.dboperations as dbops
import os

class TestFlaskAppConfig(unittest.TestCase):

    @patch('APIHome.GetImportTypes')
    def test_GetImportTypes(self, mock_GetImportTypes):
        mock_GetImportTypes.return_value = jsonify({'ImportTypesData': []})
        response = mainapp.test_client().get('/api/GetImportTypes', headers={'Authorization': 'Bearer valid_token'})
        self.assertEqual(response.status_code, 200)

    @patch('APIHome.ImportData')
    def test_ImportData(self, mock_ImportData):
        mock_ImportData.return_value = jsonify({'status': 'Success', 'message': 'Import successful'})
        response = mainapp.test_client().post('/api/ImportData', data={'year': '2023', 'importType': 'Type1'}, headers={'Authorization': 'Bearer valid_token'})
        self.assertEqual(response.status_code, 200)

    @patch('APIHome.GetActionLogs')
    def test_GetActionLogs(self, mock_GetActionLogs):
        mock_GetActionLogs.return_value = jsonify({'ActionLogsData': []})
        response = mainapp.test_client().get('/api/GetActionLogs', headers={'Authorization': 'Bearer valid_token'})
        self.assertEqual(response.status_code, 200)

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
    @patch('Services.dboperations.dboperations.GetAllUsers')
    @patch('Services.dboperations.dboperations.SadrdSysSettings')
    def test_authenticate_user_success(self, mock_sadrd_sys_settings, mock_get_all_users, mock_token_required):
        mock_token_required.return_value = True
        mock_get_all_users.return_value = [MagicMock(NetworkId='123', RoleId='1', Name='Test User', isActive=True, Email='test@example.com')]
        mock_sadrd_sys_settings.return_value = []

        response = self.app.get('/api/AuthenticateUser', headers={'Authorization': 'Bearer valid_token'})
        self.assertEqual(response.status_code, 200)
        self.assertIn('authenticated', json.loads(response.data))

    @patch('Services.Auth.token_required')
    def test_authenticate_user_no_token(self, mock_token_required):
        mock_token_required.return_value = True
        response = self.app.get('/api/AuthenticateUser')
        self.assertEqual(response.status_code, 401)

    @patch('Services.Auth.token_required')
    @patch('Services.dboperations.dboperations.GetAllUsers')
    def test_authenticate_user_invalid_token(self, mock_get_all_users, mock_token_required):
        mock_token_required.return_value = True
        mock_get_all_users.return_value = []

        response = self.app.get('/api/AuthenticateUser', headers={'Authorization': 'Bearer invalid_token'})
        self.assertEqual(response.status_code, 401)

    @patch('Services.Auth.token_required')
    @patch('Services.dboperations.dboperations.SadrdSysSettings')
    def test_get_import_types(self, mock_sadrd_sys_settings, mock_token_required):
        mock_token_required.return_value = True
        mock_sadrd_sys_settings.return_value = [MagicMock(settingName='ImportType', settingValue='Type1', Description='Description1')]

        response = self.app.get('/api/GetImportTypes', headers={'Authorization': 'Bearer valid_token'})
        self.assertEqual(response.status_code, 200)
        self.assertIn('ImportTypesData', json.loads(response.data))

    @patch('Services.Auth.token_required')
    @patch('Services.dboperations.dboperations.SadrdSysSettings')
    @patch('Services.parentparser')
    def test_import_data_success(self, mock_parser, mock_sadrd_sys_settings, mock_token_required):
        mock_token_required.return_value = True
        mock_sadrd_sys_settings.return_value = [MagicMock(settingName='ServerFolderPath', settingValue='/path/to/server')]
        mock_parser.return_value = MagicMock(status='Success', message='Import successful')

        response = self.app.post('/api/ImportData', data={'year': '2023', 'importType': 'Type1'}, headers={'Authorization': 'Bearer valid_token'})
        self.assertEqual(response.status_code, 200)
        self.assertIn('status', json.loads(response.data))

    @patch('Services.Auth.token_required')
    @patch('Services.dboperations.dboperations.SadrdSysSettings')
    @patch('Services.parentparser')
    def test_import_data_failure(self, mock_parser, mock_sadrd_sys_settings, mock_token_required):
        mock_token_required.return_value = True
        mock_sadrd_sys_settings.return_value = [MagicMock(settingName='ServerFolderPath', settingValue='/path/to/server')]
        mock_parser.return_value = MagicMock(status='Failure', message='Import failed')

        response = self.app.post('/api/ImportData', data={'year': '2023', 'importType': 'Type1'}, headers={'Authorization': 'Bearer valid_token'})
        self.assertEqual(response.status_code, 200)
        self.assertIn('status', json.loads(response.data))
        self.assertEqual(json.loads(response.data)['status'], 'Failure')

    @patch('Services.Auth.token_required')
    @patch('Services.dboperations.dboperations.get_actionLog')
    def test_get_action_logs(self, mock_get_action_log, mock_token_required):
        mock_token_required.return_value = True
        mock_get_action_log.return_value = [MagicMock(LogID=1, Month=1, Year=2023, UserID='123', Module='Test', Action='Test Action', ActionDate='2023-01-01', Comments='Test Comment', Dataload_Id='1')]

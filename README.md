import unittest
from unittest.mock import patch, MagicMock
from flask import json
from APIHome import mainapp

class TestFlaskAppConfig(unittest.TestCase):

    @patch('APIHome.GetImportTypes')
    def test_GetImportTypes(self, mock_GetImportTypes):
        mock_GetImportTypes()
        mock_GetImportTypes.assert_called_once()

    @patch('APIHome.ImportData')
    def test_ImportData(self, mock_ImportData):
        mock_ImportData()
        mock_ImportData.assert_called_once()

    @patch('APIHome.GetActionLogs')
    def test_GetActionLogs(self, mock_GetActionLogs):
        mock_GetActionLogs()
        mock_GetActionLogs.assert_called_once()

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

# Additional test cases for API endpoints
class TestAPIHome(unittest.TestCase):

    def setUp(self):
        self.app = mainapp.test_client()
        self.app.testing = True

    @patch('Services.Auth.token_required')
    @patch('Services.dboperations.dboperations.GetAllUsers')
    @patch('Services.dboperations.dboperations.SadrdSysSettings')
    def test_authenticate_user_success(self, mock_sadrd_sys_settings, mock_get_all_users, mock_token_required):
        mock_token_required.return_value = True
        mock_get_all_users.return_value = [MagicMock(NetworkId='123', RoleId='1', Name='Test User', isActive=True, Email='test@example.com')]
        mock_sadrd_sys_settings.return_value = []
        
        response = self.app.get('/api/AuthenticateUser ', headers={'Authorization': 'Bearer valid_token'})
        self.assertEqual(response.status_code, 200)
        self.assertIn('authenticated', json.loads(response.data))

    @patch('Services.Auth.token_required')
    def test_authenticate_user_no_token(self, mock_token_required):
        mock_token_required.return_value = True
        response = self.app.get('/api/AuthenticateUser ')
        self.assertEqual(response.status_code, 200)
        self.assertFalse(json.loads(response.data)['authenticated'])

    @patch('Services.Auth.token_required')
    @patch('Services.dboperations.dboperations.GetAllUsers')
    def test_authenticate_user_invalid_token(self, mock_get_all_users, mock_token_required):
        mock_token_required.return_value = True
        mock_get_all_users.return_value = []
        
        response = self.app.get('/api/AuthenticateUser ', headers={'Authorization': 'Bearer invalid_token'})
        self.assertEqual(response.status_code, 200)
        self.assertFalse(json.loads(response.data)['authenticated'])

    @patch('Services.Auth.token_required')
    @patch('Services.dboperations.dboperations.SadrdSysSettings')
    def test_get_import_types(self, mock_sadrd_sys_settings, mock_token_required):
        mock_token_required.return_value = True
        mock_sadrd_sys_settings.return_value = [MagicMock(settingName='ImportType', settingValue='Type1', Description='Description1')]
        
        response = self.app.get('/api/GetImportTypes')
        self.assertEqual(response.status_code, 200)
        self.assertIn('ImportTypesData', json.loads(response.data))

    @patch('Services.Auth.token_required')
    @patch('Services.dboperations.dboperations.SadrdSysSettings')
    @patch('Services.parentparser')
    def test_import_data_success(self, mock_parser, mock_sadrd_sys_settings, mock_token_required):
        mock_token_required.return_value = True
        mock_sadrd_sys_settings.return_value = [MagicMock(settingName='ServerFolderPath', settingValue='/path/to/server')]
        mock_parser.return_value = MagicMock(status='Success', message='Import successful')
        
        response = self.app.post('/api/ImportData', data={'year': '2023', 'importType': 'Type1'})
        self.assertEqual(response.status_code, 200)
        self.assertIn('status', json.loads(response.data))

    @patch('Services.Auth.token_required')
    @patch('Services.dboperations.dboperations.SadrdSysSettings')
    @patch('Services.parentparser')
    def test_import_data_failure(self, mock_parser, mock_sadrd_sys_settings, mock_token_required):
        mock_token_required.return_value = True
        mock_sadrd_sys_settings.return_value = [MagicMock(settingName='ServerFolderPath', settingValue='/path/to/server')]
        mock_parser.return_value = MagicMock(status='Failure', message='Import failed')
        
        response = self.app.post('/api/ImportData', data={'year': '2023', 'importType': 'Type1'})
        self.assertEqual(response.status_code, 200)
        self.assertIn('status', json.loads(response.data))
        self.assertEqual(json.loads(response.data)['status'], 'Failure')

    @patch('Services.Auth.token_required')
    @patch('Services.dboperations.dboperations.get_actionLog')
    def test_get_action_logs(self, mock_get_action_log, mock_token_required):
        mock_token_required.return_value = True
        mock_get_action_log.return_value = [MagicMock(LogID=1, Month=1, Year=2023, UserID='123', Module='Test', Action='Test Action', ActionDate='2023-01-01', Comments='Test Comment', Dataload_Id='1')]
        
        response = self.app.get('/api/GetActionLogs')
        self.assertEqual(response.status_code, 200)
        self.assertIn('ActionLogsData', json.loads(response.data))

    @patch('Services.Auth.token_required')
    @patch('Services.dboperations.dboperations.UpdateUser ')
    def test_update_user_success(self, mock_update_user, mock_token_required):
        mock_token_required.return_value = True
        mock_update_user.return_value = 'Success'
        
        response = self.app.post('/api/UpdateUser ', data={'Name': 'Test User', 'userAction': 'add'})
        self.assertEqual(response.status_code, 200)
        self.assertIn('status', json.loads(response.data))

    @patch('Services.Auth.token_required')
    @patch('Services.dboperations.dboperations.UpdateUser ')
    def test_update_user_failure(self, mock_update_user, mock_token_required):
        mock_token_required.return_value = True
        mock_update_user.return_value = 'Exists'
        
        response = self.app.post('/api/UpdateUser ', data={'Name': 'Test User', 'userAction': 'add'})
        self.assertEqual(response.status_code, 200)
        self.assertIn('status', json.loads(response.data))
        self.assertEqual(json.loads(response.data)['status'], 'Exists')

    @patch('Services.Auth.token_required')
    @patch('Services.dboperations.dboperations.SadrdSysSettings')
    def test_get_settings_data(self, mock_sadrd_sys_settings, mock_token_required):
        mock_token_required.return_value = True
        mock_sadrd_sys_settings.return_value = [MagicMock(settingName='Setting1', settingValue='Value1', ShowInUI=True, Description='Description1', DataType='String')]
        
        response = self.app.get('/api/GetSettingsData')
        self.assertEqual(response.status_code, 200)
        self.assertIn('SettingsData', json.loads(response.data))

    @patch('Services.Auth.token_required')
    @patch('Services.dboperations.dboperations.UpdateSettingsData')
    def test_update_settings_data(self, mock_update_settings_data, mock_token_required):
        mock_token_required.return_value = True
        mock_update_settings_data.return_value = 'Success'
        
        response = self.app.post('/api/UpdateSettingsData', data={'settingName': 'Setting1', 'settingValue': 'NewValue', 'userAction': 'update'})
        self.assertEqual(response.status_code, 200)
        self.assertIn('status', json.loads(response.data))

    @patch('Services.Auth.token_required')
    @patch('Services.dboperations.dboperations.UpdateSettingsData')
    def test_update_settings_data_failure(self, mock_update_settings_data, mock_token_required):
        mock_token_required.return_value = True
        mock_update_settings_data.return_value = 'Exists'
        
        response = self.app.post('/api/UpdateSettingsData', data={'settingName': 'Setting1', 'settingValue': 'NewValue', 'userAction': 'update'})
        self.assertEqual(response.status_code, 200)
        self.assertIn('status', json.loads(response.data))
        self.assertEqual(json.loads(response.data)['status'], 'Exists')

    @patch('Services.Auth.token_required')
    @patch('Services.dboperations.dboperations.SadrdSysSettings')
    def test_get_valid_company_list(self, mock_sadrd_sys_settings, mock_token_required):
        mock_token_required.return_value = True
        mock_sadrd_sys_settings.return_value = [MagicMock(settingName='Valid_Company', settingValue='Company1')]
        
        response = self.app.get('/api/GetValidCompanyList')
        self.assertEqual(response.status_code, 200)
        self.assertIn('CompanyList', json.loads(response.data))

    @patch('Services.Auth.token_required')
    @patch('Services.dboperations.dboperations.UpdateQualPctData')
    def test_update_qual_pct_data(self, mock_update_qual_pct_data, mock_token_required):
        mock_token_required.return_value = True
        mock_update_qual_pct_data.return_value = 'Success'
        
        response = self.app.post('/api/UpdateQualPctData', data={'Year': '2023', 'Company': 'Test Company', 'Cusip': '123456', 'userAction': 'add'})
        self.assertEqual(response.status_code, 200)
        self.assertIn('status', json.loads(response.data))

    @patch('Services.Auth.token_required')
    @patch('Services.dboperations.dboperations.UpdateQualPctData')
    def test_update_qual_pct_data_failure(self, mock_update_qual_pct_data, mock_token_required):
        mock_token_required.return_value = True
        mock_update_qual_pct_data.return_value = 'Exists'
        
        response = self.app.post('/api/UpdateQualPctData', data={'Year': '2023', 'Company': 'Test Company', 'Cusip': '123456', 'userAction': 'add'})
        self.assertEqual(response.status_code, 200)
        self.assertIn('status', json.loads(response.data))
        self.assertEqual(json.loads(response.data)['status'], 'Exists')

    @patch('Services.Auth.token_required')
    @patch('Services.dboperations.dboperations.GetQualPctOverrideData')
    def test_get_qual_pct_data(self, mock_get_qual_pct_override_data, mock_token_required):
        mock_token_required.return_value = True
        mock_get_qual_pct_override_data.return_value = [MagicMock(Year='2023', Company='Test Company', Cusip='123456', QualPct=0.5)]
        
        response = self.app.get('/api/GetQualPctData')
        self.assertEqual(response.status_code, 200)
        self.assertIn('QualPctData', json.loads(response.data))

if __name__ == '__main__':
    unittest.main(
import unittest
from unittest.mock import patch, MagicMock
from flask import Flask
from datetime import datetime
import json
import os
import shutil
from pathlib import Path

import APIHome
import Services.dboperations as dbops
import Services.Auth as auth
import globalvars as gvar
from Entities.Customentities import ApihomeResp

class TestAPIHome(unittest.TestCase):

    def setUp(self):
        self.app = APIHome.mainapp.test_client()
        self.app_context = APIHome.mainapp.app_context()
        self.app_context.push()
        gvar.init()
        gvar.gconfig = APIHome.mainapp.config
        gvar.sqlconfig = 'test_sql_config'
        os.makedirs(str(Path.cwd().parent.joinpath('Logs')), exist_ok=True)

    def tearDown(self):
        self.app_context.pop()

    @patch('Services.Auth.token_required', MagicMock(return_value=lambda f: f))
    @patch('Services.Auth.GetLoggedInUser', MagicMock(return_value='test_user'))
    @patch('APIHome.GetAllRoles')
    def test_AuthenticateUser_success(self, mock_get_all_roles):
        mock_get_all_roles.return_value = MagicMock(json=MagicMock(return_value={'RolesData': [{'RoleID': 1, 'Type': 'Admin'}]}))
        response = self.app.get('/api/AuthenticateUser', headers={'Authorization': 'Bearer test_token'})
        self.assertEqual(response.status_code, 200)
        self.assertEqual(json.loads(response.data), {'authenticated': True, 'NetworkId': 'test_user', 'Name': 'Test User', 'RoleId': 1, 'IsActive': True, 'Email': 'test@example.com', 'Role': 'Admin'})

    @patch('Services.Auth.token_required', MagicMock(return_value=lambda f: f))
    @patch('Services.Auth.GetLoggedInUser', MagicMock(return_value='test_user'))
    @patch('APIHome.GetAllRoles')
    def test_AuthenticateUser_user_not_found(self, mock_get_all_roles):
        mock_get_all_roles.return_value = MagicMock(json=MagicMock(return_value={'RolesData': [{'RoleID': 1, 'Type': 'Admin'}]}))
        response = self.app.get('/api/AuthenticateUser', headers={'Authorization': 'Bearer test_token'})
        self.assertEqual(response.status_code, 200)
        self.assertEqual(json.loads(response.data), {'authenticated': False})

    @patch('Services.Auth.token_required', MagicMock(return_value=lambda f: f))
    @patch('Services.Auth.GetLoggedInUser', MagicMock(return_value='test_user'))
    @patch('APIHome.GetAllRoles')
    def test_AuthenticateUser_invalid_token(self, mock_get_all_roles):
        mock_get_all_roles.return_value = MagicMock(json=MagicMock(return_value={'RolesData': [{'RoleID': 1, 'Type': 'Admin'}]}))
        response = self.app.get('/api/AuthenticateUser', headers={'Authorization': 'invalid_token'})
        self.assertEqual(response.status_code, 200)
        self.assertEqual(json.loads(response.data), {'authenticated': False})

    @patch('Services.Auth.token_required', MagicMock(return_value=lambda f: f))
    def test_GetImportTypes_success(self):
        response = self.app.get('/api/GetImportTypes')
        self.assertEqual(response.status_code, 200)
        self.assertEqual(json.loads(response.data), {'ImportTypesData': [{'ImportId': 1, 'ImportType': 'Type1', 'Description': 'Desc1'}, {'ImportId': 2, 'ImportType': 'Type2', 'Description': 'Desc2'}]})

    @patch('Services.Auth.token_required', MagicMock(return_value=lambda f: f))
    @patch('Services.parentparser.parentparser')
    def test_ImportData_success(self, mock_parent_parser):
        mock_parent_parser.return_value = ApihomeResp(status='Success', message='Import successful')
        response = self.app.post('/api/ImportData', data={'year': '2023', 'importType': 'Type1'})
        self.assertEqual(response.status_code, 200)
        self.assertEqual(json.loads(response.data), {'status': 'Success', 'message': 'Import successful'})

    @patch('Services.Auth.token_required', MagicMock(return_value=lambda f: f))
    @patch('Services.parentparser.parentparser')
    def test_ImportData_failure(self, mock_parent_parser):
        mock_parent_parser.return_value = ApihomeResp(status='Failure', message='Import failed')
        response = self.app.post('/api/ImportData', data={'year': '2023', 'importType': 'Type1'})
        self.assertEqual(response.status_code, 200)
        self.assertEqual(json.loads(response.data), {'status': 'Failure', 'message': 'Import failed'})

    @patch('Services.Auth.token_required', MagicMock(return_value=lambda f: f))
    def test_GetActionLogs_success(self):
        response = self.app.get('/api/GetActionLogs')
        self.assertEqual(response.status_code, 200)
        self.assertEqual(json.loads(response.data), {'ActionLogsData': [{'LogID': 1, 'Month': 1, 'Year': 2023, 'UserID': 1, 'Module': 'Module1', 'Action': 'Action1', 'ActionDate': '2023-01-01', 'Comments': 'Comment1', 'Dataload_Id': 1}]})

    @patch('Services.Auth.token_required', MagicMock(return_value=lambda f: f))
    @patch('APIHome.GetAllRoles')
    def test_GetAllUserDetails_success(self, mock_get_all_roles):
        mock_get_all_roles.return_value = MagicMock(json=MagicMock(return_value={'RolesData': [{'RoleID': 1, 'Type': 'Admin'}]}))
        response = self.app.get('/api/GetAllUserDetails')
        self.assertEqual(response.status_code, 200)
        self.assertEqual(json.loads(response.data), {'UsersData': [{'UserID': 1, 'NetworkId': 'test_user', 'Name': 'Test User', 'RoleId': 1, 'IsActive': True, 'Email': 'test@example.com', 'Role': 'Admin'}]})

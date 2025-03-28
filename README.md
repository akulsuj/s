import unittest
from unittest.mock import patch, MagicMock
from flask import Flask
from datetime import datetime
import json
import os
import shutil
from pathlib import Path

import APIHome
from Services.dboperations import dboperations
from Services.Auth import token_required, GetLoggedInUser
from Entities.Customentities import ApihomeResp
import globalvars as gvar
import pandas as pd

class TestAPIHome(unittest.TestCase):

    def setUp(self):
        self.app = APIHome.mainapp
        self.client = self.app.test_client()
        self.app_context = self.app.app_context()
        self.app_context.push()
        self.mock_dbops = MagicMock(spec=dboperations)
        self.mock_token_required = patch('APIHome.auth.token_required', return_value=lambda x: x).start()
        self.mock_get_logged_in_user = patch('APIHome.auth.GetLoggedInUser', return_value='testuser').start()
        self.mock_insert_server_event_log = patch('APIHome.insertServerEventLog').start()
        self.mock_parentparser = patch('APIHome.parentparser').start()
        self.mock_admin_authorize = patch('APIHome.AdminAuthorize', return_value=True).start()
        self.mock_path_exists = patch('os.path.exists', return_value=True).start()
        self.mock_os_makedirs = patch('os.makedirs').start()
        self.mock_os_listdir = patch('os.listdir', return_value=['test.xlsx']).start()
        self.mock_shutil_move = patch('shutil.move').start()

        gvar.init()
        gvar.gconfig = self.app.config
        gvar.sqlconfig = 'test_sqlconfig'
        gvar.user_id = 'test_user_id'
        gvar.user_ip_address = '127.0.0.1'
        gvar.func = 'test_func'

    def tearDown(self):
        self.mock_token_required.stop()
        self.mock_get_logged_in_user.stop()
        self.mock_insert_server_event_log.stop()
        self.mock_parentparser.stop()
        self.mock_admin_authorize.stop()
        self.mock_path_exists.stop()
        self.mock_os_makedirs.stop()
        self.mock_os_listdir.stop()
        self.mock_shutil_move.stop()
        self.app_context.pop()

    def test_AuthenticateUser_success(self):
        self.mock_dbops.GetAllUsers.return_value = [MagicMock(NetworkId='testuser', Name='Test User', RoleId=1, isActive=True, Email='test@example.com')]
        self.mock_dbops.SadrdSysSettings.return_value = [MagicMock(settingName='test', settingValue='test_value')]
        self.mock_dbops.GetAllRoles.return_value = MagicMock(json=MagicMock(return_value={'RolesData': [{'RoleID': 1, 'Type': 'Admin'}]}))
        with patch('APIHome.dbops_obj', self.mock_dbops):
            response = self.client.get('/api/AuthenticateUser', headers={'Authorization': 'Bearer test_token'})
            self.assertEqual(response.status_code, 200)
            data = json.loads(response.data)
            self.assertTrue(data['authenticated'])
            self.assertEqual(data['NetworkId'], 'testuser')

    def test_AuthenticateUser_invalid_token(self):
        self.mock_dbops.GetAllUsers.return_value = [MagicMock(NetworkId='testuser', Name='Test User', RoleId=1, isActive=True, Email='test@example.com')]
        self.mock_dbops.SadrdSysSettings.return_value = [MagicMock(settingName='test', settingValue='test_value')]
        self.mock_dbops.GetAllRoles.return_value = MagicMock(json=MagicMock(return_value={'RolesData': [{'RoleID': 1, 'Type': 'Admin'}]}))
        with patch('APIHome.dbops_obj', self.mock_dbops):
            response = self.client.get('/api/AuthenticateUser', headers={'Authorization': 'invalid_token'})
            self.assertEqual(response.status_code, 401)

    def test_GetImportTypes(self):
        self.mock_dbops.SadrdSysSettings.return_value = [MagicMock(settingName='ImportType', settingValue='Type1', Description='Desc1'), MagicMock(settingName='ImportType', settingValue='Type2', Description='Desc2')]
        with patch('APIHome.dbops_obj', self.mock_dbops):
            response = self.client.get('/api/GetImportTypes', headers={'Authorization': 'Bearer test_token'})
            self.assertEqual(response.status_code, 200)
            data = json.loads(response.data)
            self.assertEqual(len(data['ImportTypesData']), 2)

    def test_ImportData_success(self):
        self.mock_dbops.SadrdSysSettings.return_value = [MagicMock(settingName='ServerFolderPath', settingValue='/test/path')]
        self.mock_parentparser.return_value = ApihomeResp(status='Success', message='Import successful')
        with patch('APIHome.dbops_obj', self.mock_dbops):
            response = self.client.post('/api/ImportData', data={'year': '2023', 'importType': 'Type1'}, headers={'Authorization': 'Bearer test_token'})
            self.assertEqual(response.status_code, 200)
            data = json.loads(response.data)
            self.assertEqual(data['status'], 'Success')

    def test_GetActionLogs(self):
        self.mock_dbops.get_actionLog.return_value = [MagicMock(LogID=1, Month=1, Year=2023, UserID='testuser', Module='test', Action='test', ActionDate=datetime.now(), Comments='test', Dataload_Id=1)]
        with patch('APIHome.dbops_obj', self.mock_dbops):
            response = self.client.get('/api/GetActionLogs', headers={'Authorization': 'Bearer test_token'})
            self.assertEqual(response.status_code, 200)
            data = json.loads(response.data)
            self.assertEqual(len(data['ActionLogsData']), 1)

    def test_GetAllUserDetails(self):
        self.mock_dbops.GetAllUsers.return_value = [MagicMock(NetworkId='testuser', Name='Test User', isActive=True, RoleId=1, Email='test@example.com')]
        self.mock_dbops.GetAllRoles.return_value = MagicMock(json=MagicMock(return_value={'RolesData': [{'RoleID': 1, 'Type': 'Admin'}]}))
        with patch('APIHome.dbops_obj', self.mock_dbops):
            response = self.client.get('/api/GetAllUserDetails', headers={'Authorization': 'Bearer test_token'})
            self.assertEqual(response.status_code, 200)
            data = json.loads(response.data)
            self.assertEqual(len(data['UsersData']), 1)

    def test_GetAllRoles(self):
        self.mock_dbops.GetAllRoles.return_value = [MagicMock(RoleID=1, Type='Admin')]
        with patch('APIHome.dbops_obj', self.mock_dbops):
            response = self.client.get('/api/GetAllRoles', headers={'Authorization': 'Bearer test_token'})
            self.assertEqual(response.status_code, 200)
            data = json.loads(response.data)
            self.assertEqual(len(data['RolesData']), 1)

    def test_UpdateUser_success(self):
        self.mock_dbops.UpdateUser.return_value = 'Success'
        self.mock_dbops.SADRD_Sys_Message.return_value = [MagicMock(MessageNumber='E021', Message='User [Record] [UserAction] successfully.', Action='Test Action')]
        with patch('APIHome.dbops_obj', self.mock_dbops):
            response = self.client.post('/api/UpdateUser', data={'userAction': 'add', 'Name': 'Test User'}, headers={'Authorization': 'Bearer test_token'})
            self.assertEqual(response.status_

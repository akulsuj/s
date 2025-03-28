 pytest --cov . test/ --cov-report html
================================================== test session starts ==================================================
platform win32 -- Python 3.9.13, pytest-7.2.0, pluggy-1.5.0
rootdir: C:\Sujith\Projects\SADRD\FinanceIT_SADRD\API
plugins: Flask-Dance-3.2.0, cov-4.0.0
collected 99 items

test\test_APIHome.py .........FFFFFFFFFFFFFFFF                                                                     [ 25%]
test\test_SADRD_Dataparser.py .......                                                                              [ 32%]
test\test_config.py .....................                                                                          [ 53%]
test\test_db.py .                                                                                                  [ 54%]
test\test_globalvars.py .                                                                                          [ 55%]
test\Entities\test_Customentities.py ....                                                                          [ 59%]
test\Entities\test_dbormschemas.py .........                                                                       [ 68%]
test\Services\test_APIResponse.py .......                                                                          [ 75%]
test\Services\test_Auth.py ....F...                                                                                [ 83%]
test\Services\test_CustomException.py ..                                                                           [ 85%]
test\Services\test_dboperations.py .                                                                               [ 86%]
test\Services\test_fileoperations.py .......                                                                       [ 93%]
test\Services\test_logoperations.py ...                                                                            [ 96%]
test\Services\test_parentparser.py ...                                                                             [100%]

======================================================= FAILURES ======================================================== 
___________________________________ TestAPIHome.test_authenticate_user_invalid_token ____________________________________ 

self = <test.test_APIHome.TestAPIHome testMethod=test_authenticate_user_invalid_token>
mock_get_all_users = <MagicMock name='GetAllUsers' id='2017865345584'>
mock_token_required = <MagicMock name='token_required' id='2017865300624'>

    @patch('Services.Auth.token_required')
    @patch('Services.dboperations.dboperations.GetAllUsers')
    def test_authenticate_user_invalid_token(self, mock_get_all_users, mock_token_required):
        mock_token_required.return_value = True
        mock_get_all_users.return_value = []

        response = self.app.get('/api/AuthenticateUser ', headers={'Authorization': 'Bearer invalid_token'})
>       self.assertEqual(response.status_code, 200)
E       AssertionError: 404 != 200

test\test_APIHome.py:85: AssertionError
______________________________________ TestAPIHome.test_authenticate_user_no_token ______________________________________ 

self = <test.test_APIHome.TestAPIHome testMethod=test_authenticate_user_no_token>
mock_token_required = <MagicMock name='token_required' id='2017870547456'>

    @patch('Services.Auth.token_required')
    def test_authenticate_user_no_token(self, mock_token_required):
        mock_token_required.return_value = True
        response = self.app.get('/api/AuthenticateUser ')
>       self.assertEqual(response.status_code, 200)
E       AssertionError: 404 != 200

test\test_APIHome.py:75: AssertionError
______________________________________ TestAPIHome.test_authenticate_user_success _______________________________________ 

self = <test.test_APIHome.TestAPIHome testMethod=test_authenticate_user_success>
mock_sadrd_sys_settings = <MagicMock name='SadrdSysSettings' id='2017870723392'>
mock_get_all_users = <MagicMock name='GetAllUsers' id='2017870698576'>
mock_token_required = <MagicMock name='token_required' id='2017870690192'>

    @patch('Services.Auth.token_required')
    @patch('Services.dboperations.dboperations.GetAllUsers')
    @patch('Services.dboperations.dboperations.SadrdSysSettings')
    def test_authenticate_user_success(self, mock_sadrd_sys_settings, mock_get_all_users, mock_token_required):
        mock_token_required.return_value = True
        mock_get_all_users.return_value = [MagicMock(NetworkId='123', RoleId='1', Name='Test User', isActive=True, Email='test@example.com')]
        mock_sadrd_sys_settings.return_value = []

        response = self.app.get('/api/AuthenticateUser ', headers={'Authorization': 'Bearer valid_token'})
>       self.assertEqual(response.status_code, 200)
E       AssertionError: 404 != 200

test\test_APIHome.py:68: AssertionError
___________________________________________ TestAPIHome.test_get_action_logs ____________________________________________ 

self = <test.test_APIHome.TestAPIHome testMethod=test_get_action_logs>
mock_get_action_log = <MagicMock name='get_actionLog' id='2017865344816'>
mock_token_required = <MagicMock name='token_required' id='2017865298704'>

    @patch('Services.Auth.token_required')
    @patch('Services.dboperations.dboperations.get_actionLog')
    def test_get_action_logs(self, mock_get_action_log, mock_token_required):
        mock_token_required.return_value = True
        mock_get_action_log.return_value = [MagicMock(LogID=1, Month=1, Year=2023, UserID='123', Module='Test', Action='Test Action', ActionDate='2023-01-01', Comments='Test Comment', Dataload_Id='1')]

        response = self.app.get('/api/GetActionLogs')
        self.assertEqual(response.status_code, 200)
>       self.assertIn('ActionLogsData', json.loads(response.data))
E       AssertionError: 'ActionLogsData' not found in {'apirespmsg': 'Authorization Token is missing!'}

test\test_APIHome.py:131: AssertionError
___________________________________________ TestAPIHome.test_get_import_types ___________________________________________ 

self = <test.test_APIHome.TestAPIHome testMethod=test_get_import_types>
mock_sadrd_sys_settings = <MagicMock name='SadrdSysSettings' id='2017870611600'>
mock_token_required = <MagicMock name='token_required' id='2017870716016'>

    @patch('Services.Auth.token_required')
    @patch('Services.dboperations.dboperations.SadrdSysSettings')
    def test_get_import_types(self, mock_sadrd_sys_settings, mock_token_required):
        mock_token_required.return_value = True
        mock_sadrd_sys_settings.return_value = [MagicMock(settingName='ImportType', settingValue='Type1', Description='Description1')]
    
        response = self.app.get('/api/GetImportTypes')
        self.assertEqual(response.status_code, 200)
>       self.assertIn('ImportTypesData', json.loads(response.data))
E       AssertionError: 'ImportTypesData' not found in {'apirespmsg': 'Authorization Token is missing!'}

test\test_APIHome.py:96: AssertionError
__________________________________________ TestAPIHome.test_get_qual_pct_data ___________________________________________ 

self = <test.test_APIHome.TestAPIHome testMethod=test_get_qual_pct_data>
mock_get_qual_pct_override_data = <MagicMock name='GetQualPctOverrideData' id='2017865298752'>
mock_token_required = <MagicMock name='token_required' id='2017870713712'>

    @patch('Services.Auth.token_required')
    @patch('Services.dboperations.dboperations.GetQualPctOverrideData')
    def test_get_qual_pct_data(self, mock_get_qual_pct_override_data, mock_token_required):
        mock_token_required.return_value = True
        mock_get_qual_pct_override_data.return_value = [MagicMock(Year='2023', Company='Test Company', Cusip='123456', QualPct=0.5)]

        response = self.app.get('/api/GetQualPctData')
        self.assertEqual(response.status_code, 200)
>       self.assertIn('QualPctData', json.loads(response.data))
E       AssertionError: 'QualPctData' not found in {'apirespmsg': 'Authorization Token is missing!'}

test\test_APIHome.py:224: AssertionError
__________________________________________ TestAPIHome.test_get_settings_data ___________________________________________ 

self = <test.test_APIHome.TestAPIHome testMethod=test_get_settings_data>
mock_sadrd_sys_settings = <MagicMock name='SadrdSysSettings' id='2017870577872'>
mock_token_required = <MagicMock name='token_required' id='2017870552512'>

    @patch('Services.Auth.token_required')
    @patch('Services.dboperations.dboperations.SadrdSysSettings')
    def test_get_settings_data(self, mock_sadrd_sys_settings, mock_token_required):
        mock_token_required.return_value = True
        mock_sadrd_sys_settings.return_value = [MagicMock(settingName='Setting1', settingValue='Value1', ShowInUI=True, Description='Description1', DataType='String')]

        response = self.app.get('/api/GetSettingsData')
        self.assertEqual(response.status_code, 200)
>       self.assertIn('SettingsData', json.loads(response.data))
E       AssertionError: 'SettingsData' not found in {'apirespmsg': 'Authorization Token is missing!'}

test\test_APIHome.py:162: AssertionError
________________________________________ TestAPIHome.test_get_valid_company_list ________________________________________

self = <test.test_APIHome.TestAPIHome testMethod=test_get_valid_company_list>
mock_sadrd_sys_settings = <MagicMock name='SadrdSysSettings' id='2017870739104'>
mock_token_required = <MagicMock name='token_required' id='2017870709760'>

    @patch('Services.Auth.token_required')
    @patch('Services.dboperations.dboperations.SadrdSysSettings')
    def test_get_valid_company_list(self, mock_sadrd_sys_settings, mock_token_required):
        mock_token_required.return_value = True
        mock_sadrd_sys_settings.return_value = [MagicMock(settingName='Valid_Company', settingValue='Company1')]

        response = self.app.get('/api/GetValidCompanyList')
        self.assertEqual(response.status_code, 200)
>       self.assertIn('CompanyList', json.loads(response.data))
E       AssertionError: 'CompanyList' not found in {'apirespmsg': 'Authorization Token is missing!'}

test\test_APIHome.py:193: AssertionError
_________________________________________ TestAPIHome.test_import_data_failure __________________________________________ 
C:\Program Files\Python39\lib\unittest\mock.py:1333: in patched
    with self.decoration_helper(patched,
C:\Program Files\Python39\lib\contextlib.py:119: in __enter__
    return next(self.gen)
C:\Program Files\Python39\lib\unittest\mock.py:1315: in decoration_helper
    arg = exit_stack.enter_context(patching)
C:\Program Files\Python39\lib\contextlib.py:448: in enter_context
    result = _cm_type.__enter__(cm)
C:\Program Files\Python39\lib\unittest\mock.py:1404: in __enter__
    original, local = self.get_original()
_ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ 

self = <unittest.mock._patch object at 0x000001D5D1F8D700>

    def get_original(self):
        target = self.getter()
        name = self.attribute

        original = DEFAULT
        local = False

        try:
            original = target.__dict__[name]
        except (AttributeError, KeyError):
            original = getattr(target, name, DEFAULT)
        else:
            local = True

        if name in _builtins and isinstance(target, ModuleType):
            self.create = True
    
        if not self.create and original is DEFAULT:
>           raise AttributeError(
                "%s does not have the attribute %r" % (target, name)
            )
E           AttributeError: <module 'Services' from 'C:\\Sujith\\Projects\\SADRD\\FinanceIT_SADRD\\API\\Services\\__init__.py'> does not have the attribute 'parentparser'

C:\Program Files\Python39\lib\unittest\mock.py:1377: AttributeError
_________________________________________ TestAPIHome.test_import_data_success __________________________________________ 
C:\Program Files\Python39\lib\unittest\mock.py:1333: in patched
    with self.decoration_helper(patched,
C:\Program Files\Python39\lib\contextlib.py:119: in __enter__
    return next(self.gen)
C:\Program Files\Python39\lib\unittest\mock.py:1315: in decoration_helper
    arg = exit_stack.enter_context(patching)
C:\Program Files\Python39\lib\contextlib.py:448: in enter_context
    result = _cm_type.__enter__(cm)
C:\Program Files\Python39\lib\unittest\mock.py:1404: in __enter__
    original, local = self.get_original()
_ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ 

self = <unittest.mock._patch object at 0x000001D5D1F8D4C0>

    def get_original(self):
        target = self.getter()
        name = self.attribute

        original = DEFAULT
        local = False

        try:
            original = target.__dict__[name]
        except (AttributeError, KeyError):
            original = getattr(target, name, DEFAULT)
        else:
            local = True

        if name in _builtins and isinstance(target, ModuleType):
            self.create = True

        if not self.create and original is DEFAULT:
>           raise AttributeError(
                "%s does not have the attribute %r" % (target, name)
            )
E           AttributeError: <module 'Services' from 'C:\\Sujith\\Projects\\SADRD\\FinanceIT_SADRD\\API\\Services\\__init__.py'> does not have the attribute 'parentparser'

C:\Program Files\Python39\lib\unittest\mock.py:1377: AttributeError
_________________________________________ TestAPIHome.test_update_qual_pct_data _________________________________________ 

self = <test.test_APIHome.TestAPIHome testMethod=test_update_qual_pct_data>
mock_update_qual_pct_data = <MagicMock name='UpdateQualPctData' id='2017870599744'>
mock_token_required = <MagicMock name='token_required' id='2017871599648'>

    @patch('Services.Auth.token_required')
    @patch('Services.dboperations.dboperations.UpdateQualPctData')
    def test_update_qual_pct_data(self, mock_update_qual_pct_data, mock_token_required):
        mock_token_required.return_value = True
        mock_update_qual_pct_data.return_value = 'Success'

        response = self.app.post('/api/UpdateQualPctData', data={'Year': '2023', 'Company': 'Test Company', 'Cusip': '123456', 'userAction': 'add'})
        self.assertEqual(response.status_code, 200)
>       self.assertIn('status', json.loads(response.data))
E       AssertionError: 'status' not found in {'apirespmsg': 'Authorization Token is missing!'}

test\test_APIHome.py:203: AssertionError
_____________________________________ TestAPIHome.test_update_qual_pct_data_failure _____________________________________ 

self = <test.test_APIHome.TestAPIHome testMethod=test_update_qual_pct_data_failure>
mock_update_qual_pct_data = <MagicMock name='UpdateQualPctData' id='2017871434992'>
mock_token_required = <MagicMock name='token_required' id='2017870680800'>

    @patch('Services.Auth.token_required')
    @patch('Services.dboperations.dboperations.UpdateQualPctData')
    def test_update_qual_pct_data_failure(self, mock_update_qual_pct_data, mock_token_required):
        mock_token_required.return_value = True
        mock_update_qual_pct_data.return_value = 'Exists'

        response = self.app.post('/api/UpdateQualPctData', data={'Year': '2023', 'Company': 'Test Company', 'Cusip': '123456', 'userAction': 'add'})
        self.assertEqual(response.status_code, 200)
>       self.assertIn('status', json.loads(response.data))
E       AssertionError: 'status' not found in {'apirespmsg': 'Authorization Token is missing!'}

test\test_APIHome.py:213: AssertionError
_________________________________________ TestAPIHome.test_update_settings_data _________________________________________ 

self = <test.test_APIHome.TestAPIHome testMethod=test_update_settings_data>
mock_update_settings_data = <MagicMock name='UpdateSettingsData' id='2017870723488'>
mock_token_required = <MagicMock name='token_required' id='2017871109712'>

    @patch('Services.Auth.token_required')
    @patch('Services.dboperations.dboperations.UpdateSettingsData')
    def test_update_settings_data(self, mock_update_settings_data, mock_token_required):
        mock_token_required.return_value = True
        mock_update_settings_data.return_value = 'Success'

        response = self.app.post('/api/UpdateSettingsData', data={'settingName': 'Setting1', 'settingValue': 'NewValue', 'userAction': 'update'})
        self.assertEqual(response.status_code, 200)
>       self.assertIn('status', json.loads(response.data))
E       AssertionError: 'status' not found in {'apirespmsg': 'Authorization Token is missing!'}

test\test_APIHome.py:172: AssertionError
_____________________________________ TestAPIHome.test_update_settings_data_failure _____________________________________ 

self = <test.test_APIHome.TestAPIHome testMethod=test_update_settings_data_failure>
mock_update_settings_data = <MagicMock name='UpdateSettingsData' id='2017870644128'>
mock_token_required = <MagicMock name='token_required' id='2017870699776'>

    @patch('Services.Auth.token_required')
    @patch('Services.dboperations.dboperations.UpdateSettingsData')
    def test_update_settings_data_failure(self, mock_update_settings_data, mock_token_required):
        mock_token_required.return_value = True
        mock_update_settings_data.return_value = 'Exists'

        response = self.app.post('/api/UpdateSettingsData', data={'settingName': 'Setting1', 'settingValue': 'NewValue', 'userAction': 'update'})
        self.assertEqual(response.status_code, 200)
>       self.assertIn('status', json.loads(response.data))
E       AssertionError: 'status' not found in {'apirespmsg': 'Authorization Token is missing!'}

test\test_APIHome.py:182: AssertionError
_________________________________________ TestAPIHome.test_update_user_failure __________________________________________ 

self = <test.test_APIHome.TestAPIHome testMethod=test_update_user_failure>
mock_update_user = <MagicMock name='UpdateUser ' id='2017870843376'>
mock_token_required = <MagicMock name='token_required' id='2017870792112'>

    @patch('Services.Auth.token_required')
    @patch('Services.dboperations.dboperations.UpdateUser ')
    def test_update_user_failure(self, mock_update_user, mock_token_required):
        mock_token_required.return_value = True
        mock_update_user.return_value = 'Exists'

        response = self.app.post('/api/UpdateUser ', data={'Name': 'Test User', 'userAction': 'add'})
>       self.assertEqual(response.status_code, 200)
E       AssertionError: 404 != 200

test\test_APIHome.py:150: AssertionError
_________________________________________ TestAPIHome.test_update_user_success __________________________________________ 

self = <test.test_APIHome.TestAPIHome testMethod=test_update_user_success>
mock_update_user = <MagicMock name='UpdateUser ' id='2017871377216'>
mock_token_required = <MagicMock name='token_required' id='2017870898464'>

    @patch('Services.Auth.token_required')
    @patch('Services.dboperations.dboperations.UpdateUser ')
    def test_update_user_success(self, mock_update_user, mock_token_required):
        mock_token_required.return_value = True
        mock_update_user.return_value = 'Success'

        response = self.app.post('/api/UpdateUser ', data={'Name': 'Test User', 'userAction': 'add'})
>       self.assertEqual(response.status_code, 200)
E       AssertionError: 404 != 200

test\test_APIHome.py:140: AssertionError
______________________________________ Test_Auth.test_token_required_invalid_token ______________________________________ 

args = (), kwargs = {}, data = 'Bearer test_token'

    @wraps(f)
    def decorated(*args, **kwargs):
        gvar.func = f.__name__
        gvar.user_id = ''
        gvar.user_name = ''
        gvar.user_id_short = ''
        gvar.user_ip_address = ''

        logging.debug("In token_required() method :: " + gvar.func)

        if (request.headers.__contains__('Authorization')):
            data = request.headers['Authorization']
            try:
>               decrypted_token = DecryptToken(data)

Services\Auth.py:100:
_ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ 

self = <MagicMock name='DecryptToken' id='2017870978160'>, args = ('Bearer test_token',), kwargs = {}

    def __call__(self, /, *args, **kwargs):
        # can't use self in-case a function / method we are mocking uses self
        # in the signature
        self._mock_check_sig(*args, **kwargs)
        self._increment_mock_call(*args, **kwargs)
>       return self._mock_call(*args, **kwargs)

C:\Program Files\Python39\lib\unittest\mock.py:1092:
_ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ 

self = <MagicMock name='DecryptToken' id='2017870978160'>, args = ('Bearer test_token',), kwargs = {}

    def _mock_call(self, /, *args, **kwargs):
>       return self._execute_mock_call(*args, **kwargs)

C:\Program Files\Python39\lib\unittest\mock.py:1096:
_ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ 

self = <MagicMock name='DecryptToken' id='2017870978160'>, args = ('Bearer test_token',), kwargs = {}
effect = Exception('Invalid token')

    def _execute_mock_call(self, /, *args, **kwargs):
        # separate from _increment_mock_call so that awaited functions are
        # executed separately from their call, also AsyncMock overrides this method

        effect = self.side_effect
        if effect is not None:
            if _is_exception(effect):
>               raise effect
E               Exception: Invalid token

C:\Program Files\Python39\lib\unittest\mock.py:1151: Exception

During handling of the above exception, another exception occurred:

self = <test.Services.test_Auth.Test_Auth testMethod=test_token_required_invalid_token>
mock_insert_actionLog = <MagicMock name='insert_actionLog' id='2017871657856'>
mock_decrypt_token = <MagicMock name='DecryptToken' id='2017870978160'>

    @patch('Services.Auth.DecryptToken')
    @patch('Services.dboperations.dboperations.insert_actionLog')
    def test_token_required_invalid_token(self, mock_insert_actionLog, mock_decrypt_token):
        mock_decrypt_token.side_effect = Exception('Invalid token')

        @auth.token_required
        def test_route():
            return jsonify({'message': 'Success'})

        with self.app.test_request_context(headers={'Authorization': 'Bearer test_token'}):
>           response = test_route()

test\Services\test_Auth.py:98:
_ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ 
Services\Auth.py:119: in decorated
    dbops_obj = dbops.dboperations()
_ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ 

self = <Services.dboperations.dboperations object at 0x000001D5D286FCD0>, newsyssession = None

    def __init__(self, newsyssession=None):
        super().__init__()

        self.params = gvar.sqlconfig
>       self.engine = sqlalchemy.create_engine(gvar.gconfig["SQLALCHEMYODBC"] % self.params,fast_executemany=True)        
E       TypeError: string indices must be integers

Services\dboperations.py:27: TypeError
=================================================== warnings summary ==================================================== 
venv\lib\site-packages\flask_sqlalchemy\__init__.py:14
venv\lib\site-packages\flask_sqlalchemy\__init__.py:14
  C:\Sujith\Projects\SADRD\FinanceIT_SADRD\API\venv\lib\site-packages\flask_sqlalchemy\__init__.py:14: DeprecationWarning: '_app_ctx_stack' is deprecated and will be removed in Flask 2.3.
    from flask import _app_ctx_stack, abort, current_app, request

venv\lib\site-packages\pandas\compat\numpy\__init__.py:10
  C:\Sujith\Projects\SADRD\FinanceIT_SADRD\API\venv\lib\site-packages\pandas\compat\numpy\__init__.py:10: DeprecationWarning: distutils Version classes are deprecated. Use packaging.version instead.
    _nlv = LooseVersion(_np_version)

venv\lib\site-packages\pandas\compat\numpy\__init__.py:11
  C:\Sujith\Projects\SADRD\FinanceIT_SADRD\API\venv\lib\site-packages\pandas\compat\numpy\__init__.py:11: DeprecationWarning: distutils Version classes are deprecated. Use packaging.version instead.
    np_version_under1p17 = _nlv < LooseVersion("1.17")

venv\lib\site-packages\pandas\compat\numpy\__init__.py:12
  C:\Sujith\Projects\SADRD\FinanceIT_SADRD\API\venv\lib\site-packages\pandas\compat\numpy\__init__.py:12: DeprecationWarning: distutils Version classes are deprecated. Use packaging.version instead.
    np_version_under1p18 = _nlv < LooseVersion("1.18")

venv\lib\site-packages\pandas\compat\numpy\__init__.py:13
  C:\Sujith\Projects\SADRD\FinanceIT_SADRD\API\venv\lib\site-packages\pandas\compat\numpy\__init__.py:13: DeprecationWarning: distutils Version classes are deprecated. Use packaging.version instead.
    _np_version_under1p19 = _nlv < LooseVersion("1.19")

venv\lib\site-packages\pandas\compat\numpy\__init__.py:14
  C:\Sujith\Projects\SADRD\FinanceIT_SADRD\API\venv\lib\site-packages\pandas\compat\numpy\__init__.py:14: DeprecationWarning: distutils Version classes are deprecated. Use packaging.version instead.
    _np_version_under1p20 = _nlv < LooseVersion("1.20")

venv\lib\site-packages\setuptools\_distutils\version.py:337
  C:\Sujith\Projects\SADRD\FinanceIT_SADRD\API\venv\lib\site-packages\setuptools\_distutils\version.py:337: DeprecationWarning: distutils Version classes are deprecated. Use packaging.version instead.
    other = LooseVersion(other)

venv\lib\site-packages\pandas\compat\numpy\function.py:120
venv\lib\site-packages\pandas\compat\numpy\function.py:120
  C:\Sujith\Projects\SADRD\FinanceIT_SADRD\API\venv\lib\site-packages\pandas\compat\numpy\function.py:120: DeprecationWarning: distutils Version classes are deprecated. Use packaging.version instead.
    if LooseVersion(__version__) >= LooseVersion("1.17.0"):

-- Docs: https://docs.pytest.org/en/stable/how-to/capture-warnings.html

---------- coverage: platform win32, python 3.9.13-final-0 -----------
Coverage HTML written to dir htmlcov

================================================ short test summary info ================================================ 
FAILED test/test_APIHome.py::TestAPIHome::test_authenticate_user_invalid_token - AssertionError: 404 != 200
FAILED test/test_APIHome.py::TestAPIHome::test_authenticate_user_no_token - AssertionError: 404 != 200
FAILED test/test_APIHome.py::TestAPIHome::test_authenticate_user_success - AssertionError: 404 != 200
FAILED test/test_APIHome.py::TestAPIHome::test_get_action_logs - AssertionError: 'ActionLogsData' not found in {'apirespmsg': 'Authorization Token is missing!'}
FAILED test/test_APIHome.py::TestAPIHome::test_get_import_types - AssertionError: 'ImportTypesData' not found in {'apirespmsg': 'Authorization Token is missing!'}
FAILED test/test_APIHome.py::TestAPIHome::test_get_qual_pct_data - AssertionError: 'QualPctData' not found in {'apirespmsg': 'Authorization Token is missing!'}
FAILED test/test_APIHome.py::TestAPIHome::test_get_settings_data - AssertionError: 'SettingsData' not found in {'apirespmsg': 'Authorization Token is missing!'}
FAILED test/test_APIHome.py::TestAPIHome::test_get_valid_company_list - AssertionError: 'CompanyList' not found in {'apirespmsg': 'Authorization Token is missing!'}
FAILED test/test_APIHome.py::TestAPIHome::test_import_data_failure - AttributeError: <module 'Services' from 'C:\\Sujith\\Projects\\SADRD\\FinanceIT_SADRD\\API\\Services\\__init__.py'> d...
FAILED test/test_APIHome.py::TestAPIHome::test_import_data_success - AttributeError: <module 'Services' from 'C:\\Sujith\\Projects\\SADRD\\FinanceIT_SADRD\\API\\Services\\__init__.py'> d...
FAILED test/test_APIHome.py::TestAPIHome::test_update_qual_pct_data - AssertionError: 'status' not found in {'apirespmsg': 'Authorization Token is missing!'}
FAILED test/test_APIHome.py::TestAPIHome::test_update_qual_pct_data_failure - AssertionError: 'status' not found in {'apirespmsg': 'Authorization Token is missing!'}
FAILED test/test_APIHome.py::TestAPIHome::test_update_settings_data - AssertionError: 'status' not found in {'apirespmsg': 'Authorization Token is missing!'}
FAILED test/test_APIHome.py::TestAPIHome::test_update_settings_data_failure - AssertionError: 'status' not found in {'apirespmsg': 'Authorization Token is missing!'}
FAILED test/test_APIHome.py::TestAPIHome::test_update_user_failure - AssertionError: 404 != 200
FAILED test/test_APIHome.py::TestAPIHome::test_update_user_success - AssertionError: 404 != 200
FAILED test/Services/test_Auth.py::Test_Auth::test_token_required_invalid_token - TypeError: string indices must be integers
====================================== 17 failed, 82 passed, 10 warnings in 9.41s ======================================= 
PS C:\Sujith\Projects\SADRD\FinanceIT_SADRD\API>















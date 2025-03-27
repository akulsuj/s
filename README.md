pytest --cov . test/ --cov-report html
===================================================== test session starts =====================================================
platform win32 -- Python 3.9.13, pytest-7.2.0, pluggy-1.5.0
rootdir: C:\Sujith\Projects\SADRD\FinanceIT_SADRD\API
plugins: Flask-Dance-3.2.0, cov-4.0.0
collected 86 items

test\test_APIHome.py FFFFFFF.FFFF                                                                                        [ 13%]
test\test_config.py .....................                                                                                [ 38%]
test\test_db.py .                                                                                                        [ 39%]
test\test_globalvars.py .                                                                                                [ 40%] 
test\Entities\test_Customentities.py ....                                                                                [ 45%]
test\Entities\test_dbormschemas.py .........                                                                             [ 55%]
test\Services\test_APIResponse.py .......                                                                                [ 63%]
test\Services\test_Auth.py ....F...                                                                                      [ 73%]
test\Services\test_CustomException.py ..                                                                                 [ 75%] 
test\Services\test_SADRD_CLI.py .......                                                                                  [ 83%]
test\Services\test_dboperations.py .                                                                                     [ 84%]
test\Services\test_fileoperations.py .......                                                                             [ 93%]
test\Services\test_logoperations.py ...                                                                                  [ 96%]
test\Services\test_parentparser.py ...                                                                                   [100%]

========================================================== FAILURES =========================================================== 
_________________________________________ TestAPIHome.test_authenticate_user_no_token _________________________________________ 

self = <test.test_APIHome.TestAPIHome testMethod=test_authenticate_user_no_token>

    def test_authenticate_user_no_token(self):
        response = self.client.get("/api/AuthenticateUser")
        self.assertEqual(response.status_code, 200)
        data = response.get_json()
>       self.assertFalse(data.get("authenticated", True))
E       AssertionError: True is not false

test\test_APIHome.py:68: AssertionError
_________________________________________ TestAPIHome.test_authenticate_user_success __________________________________________ 

args = (), kwargs = {}
data = 'Bearer eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJzdWIiOiJ0ZXN0dXNlciJ9.t7LN7KC_hux73HoGflGcQbNaswBhh2wBJ8oNB0bs7m4'       
decrypted_token = {'sub': 'testuser'}

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
                decrypted_token = DecryptToken(data)
                gvar.user_id_short = GetLoggedInUser(data)
>               gvar.user_name = str(decrypted_token['name'])
E               KeyError: 'name'

Services\Auth.py:102: KeyError

During handling of the above exception, another exception occurred:

self = <test.test_APIHome.TestAPIHome testMethod=test_authenticate_user_success>
mock_get_all_roles = <MagicMock name='GetAllRoles' id='2530040179632'>
mock_get_logged_in_user = <MagicMock name='GetLoggedInUser' id='2530040186576'>

    @patch("APIHome.auth.GetLoggedInUser")
    @patch("APIHome.GetAllRoles")
    def test_authenticate_user_success(self, mock_get_all_roles, mock_get_logged_in_user):
        mock_get_logged_in_user.return_value = "testuser"
        APIHome.gvar.sadrdUsersList = [Mock(NetworkId="testuser", Name="Test User", RoleId=1, isActive=True, Email="test@example.com")]
        mock_get_all_roles.return_value.json.get.return_value = [{"RoleID": 1, "Type": "Admin"}]

        token = self.generate_jwt_token()
>       response = self.client.get("/api/AuthenticateUser", headers={"Authorization": f"Bearer {token}"})

test\test_APIHome.py:59:
_ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ 
venv\lib\site-packages\werkzeug\test.py:1141: in get
    return self.open(*args, **kw)
venv\lib\site-packages\flask\testing.py:238: in open
    response = super().open(
venv\lib\site-packages\werkzeug\test.py:1095: in open
    response = self.run_wsgi_app(request.environ, buffered=buffered)
venv\lib\site-packages\werkzeug\test.py:962: in run_wsgi_app
    rv = run_wsgi_app(self.application, environ, buffered=buffered)
venv\lib\site-packages\werkzeug\test.py:1243: in run_wsgi_app
    app_rv = app(environ, start_response)
venv\lib\site-packages\flask\app.py:2552: in __call__
    return self.wsgi_app(environ, start_response)
venv\lib\site-packages\flask\app.py:2532: in wsgi_app
    response = self.handle_exception(e)
venv\lib\site-packages\flask_cors\extension.py:165: in wrapped_function
    return cors_after_request(app.make_response(f(*args, **kwargs)))
venv\lib\site-packages\flask\app.py:2529: in wsgi_app
    response = self.full_dispatch_request()
venv\lib\site-packages\flask\app.py:1825: in full_dispatch_request
    rv = self.handle_user_exception(e)
venv\lib\site-packages\flask_cors\extension.py:165: in wrapped_function
    return cors_after_request(app.make_response(f(*args, **kwargs)))
venv\lib\site-packages\flask\app.py:1823: in full_dispatch_request
    rv = self.dispatch_request()
venv\lib\site-packages\flask\app.py:1799: in dispatch_request
    return self.ensure_sync(self.view_functions[rule.endpoint])(**view_args)
Services\Auth.py:119: in decorated
    dbops_obj = dbops.dboperations()
_ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ 

self = <Services.dboperations.dboperations object at 0x0000024D126E6820>, newsyssession = None

    def __init__(self, newsyssession=None):
        super().__init__()

        self.params = gvar.sqlconfig
>       self.engine = sqlalchemy.create_engine(gvar.gconfig["SQLALCHEMYODBC"] % self.params,fast_executemany=True)
E       TypeError: not all arguments converted during string formatting

Services\dboperations.py:27: TypeError
___________________________________________ TestAPIHome.test_close_tax_year_success ___________________________________________ 

self = <test.test_APIHome.TestAPIHome testMethod=test_close_tax_year_success>
mock_move = <MagicMock name='move' id='2530047756176'>, mock_listdir = <MagicMock name='listdir' id='2530047654880'>
mock_makedirs = <MagicMock name='makedirs' id='2530045138160'>, mock_exists = <MagicMock name='exists' id='2530045108816'>      

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
        data = response.get_json()
>       self.assertEqual(data.get("status", ""), "Success")
E       AssertionError: '' != 'Success'
E       + Success

test\test_APIHome.py:177: AssertionError
_________________________________________ TestAPIHome.test_cusip_mapping_data_success _________________________________________ 

self = <test.test_APIHome.TestAPIHome testMethod=test_cusip_mapping_data_success>
mock_parentparser = <MagicMock name='parentparser' id='2530045184800'>

    @patch("Services.parentparser.parentparser")
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
        data = response.get_json()
>       self.assertEqual(data.get("status", ""), "Success")
E       AssertionError: '' != 'Success'
E       + Success

test\test_APIHome.py:159: AssertionError
__________________________________________ TestAPIHome.test_get_action_logs_success ___________________________________________ 

self = <test.test_APIHome.TestAPIHome testMethod=test_get_action_logs_success>

    def test_get_action_logs_success(self):
        APIHome.dbops_obj.get_actionLog.return_value = [
            Mock(LogID=1, Month=3, Year=2025, UserID="testuser", Module="Test", Action="TestAction", ActionDate="2025-03-27", Comments="Comment", Dataload_Id=1)
        ]
        response = self.client.get("/api/GetActionLogs")
        self.assertEqual(response.status_code, 200)
        data = response.get_json()
>       self.assertEqual(len(data.get("ActionLogsData", [])), 1)
E       AssertionError: 0 != 1

test\test_APIHome.py:115: AssertionError
___________________________________________ TestAPIHome.test_get_all_roles_success ____________________________________________ 

self = <test.test_APIHome.TestAPIHome testMethod=test_get_all_roles_success>

    def test_get_all_roles_success(self):
        APIHome.dbops_obj.GetAllRoles.return_value = [Mock(RoleID=1, Type="Admin")]
        response = self.client.get("/api/GetAllRoles")
        self.assertEqual(response.status_code, 200)
        data = response.get_json()
>       self.assertEqual(len(data.get("RolesData", [])), 1)
E       AssertionError: 0 != 1

test\test_APIHome.py:135: AssertionError
________________________________________ TestAPIHome.test_get_all_user_details_success ________________________________________ 

self = <test.test_APIHome.TestAPIHome testMethod=test_get_all_user_details_success>
mock_get_all_roles = <MagicMock name='GetAllRoles' id='2530044912016'>

    @patch("APIHome.GetAllRoles")
    def test_get_all_user_details_success(self, mock_get_all_roles):
        APIHome.dbops_obj.GetAllUsers.return_value = [
            Mock(NetworkId="user1", Name="User One", isActive=True, RoleId=1, Email="user1@example.com")
        ]
        mock_get_all_roles.return_value.json.get.return_value = [{"RoleID": 1, "Type": "Admin"}]
        response = self.client.get("/api/GetAllUserDetails")
        self.assertEqual(response.status_code, 200)
        data = response.get_json()
>       self.assertEqual(len(data.get("UsersData", [])), 1)
E       AssertionError: 0 != 1

test\test_APIHome.py:127: AssertionError
__________________________________________ TestAPIHome.test_get_import_types_success __________________________________________ 

self = <test.test_APIHome.TestAPIHome testMethod=test_get_import_types_success>

    def test_get_import_types_success(self):
        APIHome.gvar.sadrd_settings = [
            Mock(settingName="ImportType", settingValue="Type1", Description="Desc1"),
            Mock(settingName="Other", settingValue="Value", Description="OtherDesc")
        ]
        response = self.client.get("/api/GetImportTypes")
        self.assertEqual(response.status_code, 200)
        data = response.get_json()
>       self.assertEqual(len(data.get("ImportTypesData", [])), 1)
E       AssertionError: 0 != 1

test\test_APIHome.py:79: AssertionError
___________________________________________ TestAPIHome.test_import_data_exception ____________________________________________ 

self = <test.test_APIHome.TestAPIHome testMethod=test_import_data_exception>
mock_parentparser = <MagicMock name='parentparser' id='2530045135936'>

    @patch("Services.parentparser.parentparser")
    def test_import_data_exception(self, mock_parentparser):
        APIHome.gvar.sadrd_settings = [Mock(settingName="ServerFolderPath", settingValue="/mock/path")]
        mock_parentparser.side_effect = Exception("Parser Error")
        response = self.client.post("/api/ImportData", data={"year": "2023", "importType": "testType"})
        self.assertEqual(response.status_code, 200)
        data = response.get_json()
>       self.assertEqual(data.get("status", ""), "Failure")
E       AssertionError: '' != 'Failure'
E       + Failure

test\test_APIHome.py:105: AssertionError
____________________________________________ TestAPIHome.test_import_data_success _____________________________________________ 

self = <test.test_APIHome.TestAPIHome testMethod=test_import_data_success>
mock_parentparser = <MagicMock name='parentparser' id='2530045450752'>

    @patch("Services.parentparser.parentparser")
    def test_import_data_success(self, mock_parentparser):
        APIHome.gvar.sadrd_settings = [Mock(settingName="ServerFolderPath", settingValue="/mock/path")]
        mock_parentparser.return_value = APIHome.ApihomeResp(status="Success", message="Imported successfully")
        response = self.client.post("/api/ImportData", data={"year": "2023", "importType": "testType"})
        self.assertEqual(response.status_code, 200)
        data = response.get_json()
>       self.assertEqual(data.get("status", ""), "Success")
E       AssertionError: '' != 'Success'
E       + Success

test\test_APIHome.py:96: AssertionError
____________________________________________ TestAPIHome.test_update_user_success _____________________________________________ 

self = <test.test_APIHome.TestAPIHome testMethod=test_update_user_success>

    def test_update_user_success(self):
        APIHome.dbops_obj.UpdateUser.return_value = "Success"
        APIHome.gvar.sadrd_ErrMessages = [Mock(MessageNumber="E021", Message="[Record] [UserAction] successfully", Action="")]  
        response = self.client.post("/api/UpdateUser", data={"userAction": "add", "Name": "New User"})
        self.assertEqual(response.status_code, 200)
        data = response.get_json()
>       self.assertEqual(data.get("status", ""), "Success")
E       AssertionError: '' != 'Success'
E       + Success

test\test_APIHome.py:144: AssertionError
_________________________________________ Test_Auth.test_token_required_invalid_token _________________________________________ 

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
_ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ 

self = <MagicMock name='DecryptToken' id='2530049290928'>, args = ('Bearer test_token',), kwargs = {}

    def __call__(self, /, *args, **kwargs):
        # can't use self in-case a function / method we are mocking uses self
        # in the signature
        self._mock_check_sig(*args, **kwargs)
        self._increment_mock_call(*args, **kwargs)
>       return self._mock_call(*args, **kwargs)

C:\Program Files\Python39\lib\unittest\mock.py:1092:
_ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ 

self = <MagicMock name='DecryptToken' id='2530049290928'>, args = ('Bearer test_token',), kwargs = {}

    def _mock_call(self, /, *args, **kwargs):
>       return self._execute_mock_call(*args, **kwargs)

C:\Program Files\Python39\lib\unittest\mock.py:1096:
_ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ 

self = <MagicMock name='DecryptToken' id='2530049290928'>, args = ('Bearer test_token',), kwargs = {}
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
mock_insert_actionLog = <MagicMock name='insert_actionLog' id='2530049178736'>
mock_decrypt_token = <MagicMock name='DecryptToken' id='2530049290928'>

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
_ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ 
Services\Auth.py:119: in decorated
    dbops_obj = dbops.dboperations()
_ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ 

self = <Services.dboperations.dboperations object at 0x0000024D12AFA550>, newsyssession = None

    def __init__(self, newsyssession=None):
        super().__init__()

        self.params = gvar.sqlconfig
>       self.engine = sqlalchemy.create_engine(gvar.gconfig["SQLALCHEMYODBC"] % self.params,fast_executemany=True)
E       TypeError: string indices must be integers

Services\dboperations.py:27: TypeError
====================================================== warnings summary ======================================================= 
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

=================================================== short test summary info =================================================== 
FAILED test/test_APIHome.py::TestAPIHome::test_authenticate_user_no_token - AssertionError: True is not false
FAILED test/test_APIHome.py::TestAPIHome::test_authenticate_user_success - TypeError: not all arguments converted during string formatting
FAILED test/test_APIHome.py::TestAPIHome::test_close_tax_year_success - AssertionError: '' != 'Success'
FAILED test/test_APIHome.py::TestAPIHome::test_cusip_mapping_data_success - AssertionError: '' != 'Success'
FAILED test/test_APIHome.py::TestAPIHome::test_get_action_logs_success - AssertionError: 0 != 1
FAILED test/test_APIHome.py::TestAPIHome::test_get_all_roles_success - AssertionError: 0 != 1
FAILED test/test_APIHome.py::TestAPIHome::test_get_all_user_details_success - AssertionError: 0 != 1
FAILED test/test_APIHome.py::TestAPIHome::test_get_import_types_success - AssertionError: 0 != 1
FAILED test/test_APIHome.py::TestAPIHome::test_import_data_exception - AssertionError: '' != 'Failure'
FAILED test/test_APIHome.py::TestAPIHome::test_import_data_success - AssertionError: '' != 'Success'
FAILED test/test_APIHome.py::TestAPIHome::test_update_user_success - AssertionError: '' != 'Success'
FAILED test/Services/test_Auth.py::Test_Auth::test_token_required_invalid_token - TypeError: string indices must be integers    
========================================= 12 failed, 74 passed, 10 warnings in 7.00s ==========================================
PS C:\Sujith\Projects\SADRD\FinanceIT_SADRD\API> 

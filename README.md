pytest --cov . test/ --cov-report html
===================================================== test session starts =====================================================
platform win32 -- Python 3.9.13, pytest-7.2.0, pluggy-1.5.0
rootdir: C:\Sujith\Projects\SADRD\FinanceIT_SADRD\API
plugins: Flask-Dance-3.2.0, cov-4.0.0
collected 85 items

test\test_APIHome.py FF.........                                                                                         [ 12%]
test\test_config.py .....................                                                                                [ 37%]
test\test_db.py .                                                                                                        [ 38%]
test\test_globalvars.py .                                                                                                [ 40%] 
test\Entities\test_Customentities.py ....                                                                                [ 44%]
test\Entities\test_dbormschemas.py .........                                                                             [ 55%]
test\Services\test_APIResponse.py .......                                                                                [ 63%]
test\Services\test_Auth.py ....F...                                                                                      [ 72%]
test\Services\test_CustomException.py ..                                                                                 [ 75%] 
test\Services\test_SADRD_CLI.py .......                                                                                  [ 83%]
test\Services\test_dboperations.py .                                                                                     [ 84%]
test\Services\test_fileoperations.py .......                                                                             [ 92%]
test\Services\test_logoperations.py ...                                                                                  [ 96%]
test\Services\test_parentparser.py ...                                                                                   [100%]

========================================================== FAILURES =========================================================== 
_________________________________________ TestAPIHome.test_authenticate_user_no_token _________________________________________ 

self = <test.test_APIHome.TestAPIHome testMethod=test_authenticate_user_no_token>
mock_dbops = <MagicMock name='dboperations' id='2734128965952'>
mock_create_engine = <MagicMock name='create_engine' id='2734128978096'>

    @patch("sqlalchemy.create_engine")
    @patch("Services.dboperations.dboperations")
    def test_authenticate_user_no_token(self, mock_dbops, mock_create_engine):
        mock_create_engine.return_value = Mock()
        mock_dbops.return_value = Mock()
        response = self.client.get("/api/AuthenticateUser")
        self.assertEqual(response.status_code, 200)
        data = response.get_json() or {}
        # Adjust based on actual behavior (seems to return True without token)
>       self.assertTrue(data.get("authenticated", False), "Endpoint returns authenticated=True without token")
E       AssertionError: False is not true : Endpoint returns authenticated=True without token

test\test_APIHome.py:81: AssertionError
_________________________________________ TestAPIHome.test_authenticate_user_success __________________________________________ 

self = <test.test_APIHome.TestAPIHome testMethod=test_authenticate_user_success>
mock_dbops = <MagicMock name='dboperations' id='2734129555440'>
mock_create_engine = <MagicMock name='create_engine' id='2734129065744'>
mock_get_all_roles = <MagicMock name='GetAllRoles' id='2734129611632'>
mock_get_logged_in_user = <MagicMock name='GetLoggedInUser' id='2734129635968'>

    @patch("APIHome.auth.GetLoggedInUser")
    @patch("APIHome.GetAllRoles")
    @patch("sqlalchemy.create_engine")
    @patch("Services.dboperations.dboperations")
    def test_authenticate_user_success(self, mock_dbops, mock_create_engine, mock_get_all_roles, mock_get_logged_in_user):      
        mock_create_engine.return_value = Mock()
        mock_dbops.return_value = Mock()  # Mock dboperations instance
        mock_get_logged_in_user.return_value = "testuser"
        APIHome.gvar.sadrdUsersList = [Mock(NetworkId="testuser", Name="Test User", RoleId=1, isActive=True, Email="test@example.com")]
        mock_get_all_roles.return_value.json.get.return_value = [{"RoleID": 1, "Type": "Admin"}]

        token = self.generate_jwt_token()
        response = self.client.get("/api/AuthenticateUser", headers={"Authorization": f"Bearer {token}"})
        self.assertEqual(response.status_code, 200)
        data = response.get_json() or {}
>       self.assertTrue(data.get("authenticated", False))
E       AssertionError: False is not true

test\test_APIHome.py:70: AssertionError
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

self = <MagicMock name='DecryptToken' id='2734130294592'>, args = ('Bearer test_token',), kwargs = {}

    def __call__(self, /, *args, **kwargs):
        # can't use self in-case a function / method we are mocking uses self
        # in the signature
        self._mock_check_sig(*args, **kwargs)
        self._increment_mock_call(*args, **kwargs)
>       return self._mock_call(*args, **kwargs)

C:\Program Files\Python39\lib\unittest\mock.py:1092:
_ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ 

self = <MagicMock name='DecryptToken' id='2734130294592'>, args = ('Bearer test_token',), kwargs = {}

    def _mock_call(self, /, *args, **kwargs):
>       return self._execute_mock_call(*args, **kwargs)

C:\Program Files\Python39\lib\unittest\mock.py:1096:
_ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ _ 

self = <MagicMock name='DecryptToken' id='2734130294592'>, args = ('Bearer test_token',), kwargs = {}
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
mock_insert_actionLog = <MagicMock name='insert_actionLog' id='2734130096448'>
mock_decrypt_token = <MagicMock name='DecryptToken' id='2734130294592'>

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

self = <Services.dboperations.dboperations object at 0x0000027C96CAAD30>, newsyssession = None

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
FAILED test/test_APIHome.py::TestAPIHome::test_authenticate_user_no_token - AssertionError: False is not true : Endpoint returns authenticated=True without token
FAILED test/test_APIHome.py::TestAPIHome::test_authenticate_user_success - AssertionError: False is not true
FAILED test/Services/test_Auth.py::Test_Auth::test_token_required_invalid_token - TypeError: string indices must be integers    
========================================== 3 failed, 82 passed, 10 warnings in 6.34s ========================================== 
PS C:\Sujith\Projects\SADRD\FinanceIT_SADRD\API> 

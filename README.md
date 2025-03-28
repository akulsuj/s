import unittest
from unittest.mock import patch, MagicMock, call
import pandas as pd
from sqlalchemy.exc import SQLAlchemyError
from datetime import datetime
from Services.dboperations import dboperations, TranInprogress
import globalvars as gvar
import urllib
import Entities.dbormschemas as dbsch
import logging

class TestDbOperations(unittest.TestCase):

    @patch.object(dboperations, '__init__', return_value=None)
    def setUp(self, mock_db_init):
        self.db_ops = dboperations()
        self.mock_session = MagicMock()
        self.db_ops.session = self.mock_session
        self.db_ops.metadata = MagicMock()

        # Replace with your actual DSN
        nir_dsn = "your_nir_dsn"  # Replace with your NIR DSN
        sadrd_dsn = "your_sadrd_dsn" # Replace with your SADRD DSN

        # Construct a valid URL string using the DSN
        nir_url = f"mssql+pyodbc://{nir_dsn}"
        sadrd_url = f"mssql+pyodbc://{sadrd_dsn}"

        # Mock gvar.gconfig as a dictionary with specific return values for keys
        gvar.gconfig = MagicMock()
        gvar.gconfig.__getitem__.side_effect = lambda key: {
            "DRIVER": "{test_driver}",
            "NIR_DATABASE_SERVER": "test_server",
            "NIR_DATABASE_NAME": "test_db",
            "NIR_DATABASE_AUTH_STRING": "test_auth",
            "SQLALCHEMYODBC": "mssql+pyodbc:///?odbc_connect=%s",
            "SADRD_DATABASE_SERVER": "sadrd_server",
            "SADRD_DATABASE_NAME": "sadrd_db",
            "CONNECTION_AUTH_STRING": "sadrd_auth"
        }[key]

        # Mock urllib.parse.quote_plus to return the valid url
        gvar.sqlconfig = MagicMock()
        gvar.sqlconfig.return_value = nir_url
        self.nir_url = nir_url
        self.sadrd_url = sadrd_url

        gvar.sadrd_ErrMessages = [MagicMock(MessageNumber='E019', Message='[YYYY]')]
        gvar.sadrd_settings = [MagicMock(settingName='test', settingValue='test')]
        gvar.scheduleEList = [MagicMock(Cusip='test', CusipName='test')]
        gvar.qualPctList = [MagicMock(Year=2023, Company='test', Cusip='test')]
        gvar.COMPLETED = 'completed'
        gvar.FAILED = 'failed'
        gvar.INPROGRESS = 'inprogress'
        gvar.user_id = 'test_user'

    def test_truncatetable(self):
        with patch.object(self.db_ops.session, 'execute') as mock_execute:
            self.db_ops.truncatetable('test_table')
            mock_execute.assert_called_once()
            mock_execute.reset_mock()
            self.db_ops.insert_actionLog = MagicMock()
            mock_execute.side_effect = SQLAlchemyError('test')
            with self.assertRaises(SQLAlchemyError):
                self.db_ops.truncatetable('test_table')

    def test_insert_dataloadkey(self):
        with patch.object(self.db_ops.session, 'query') as mock_query, \
             patch.object(self.db_ops.session, 'add') as mock_add, \
             patch.object(self.db_ops.session, 'commit') as mock_commit:
            mock_query.return_value.filter_by.return_value.first.return_value = None
            self.db_ops.insert_dataloadkey('loadkey', '{}', 'src')
            mock_add.assert_called_once()
            mock_commit.assert_called_once()
            mock_query.return_value.filter_by.return_value.first.return_value = MagicMock(dataload_status = gvar.INPROGRESS)
            self.db_ops.insert_actionLog = MagicMock()
            with self.assertRaises(TranInprogress):
                self.db_ops.insert_dataloadkey('loadkey', '{}', 'src')
            mock_query.return_value.filter_by.return_value.first.return_value = None
            mock_add.side_effect = SQLAlchemyError('test')
            with self.assertRaises(SQLAlchemyError):
                self.db_ops.insert_dataloadkey('loadkey', '{}', 'src')

    def test_get_dataloadkey(self):
        with patch.object(self.db_ops.session, 'query') as mock_query:
            self.db_ops.get_dataloadkey('loadkey')
            mock_query.return_value.filter_by.return_value.first.assert_called_once()
            mock_query.return_value.filter_by.return_value.first.side_effect = SQLAlchemyError('test')
            self.db_ops.insert_actionLog = MagicMock()
            self.db_ops.get_dataloadkey('loadkey')

    def test_update_dataloadkey(self):
        with patch.object(self.db_ops.session, 'commit') as mock_commit:
            self.db_ops.sysdlrec = MagicMock()
            self.db_ops.update_dataloadkey('status', 'details')
            mock_commit.assert_called_once()
            mock_commit.side_effect = SQLAlchemyError('test')
            with self.assertRaises(SQLAlchemyError):
                self.db_ops.update_dataloadkey('status', 'details')

    def test_executeSADRD_SP(self):
        with patch.object(self.db_ops.connection.cursor(), 'execute', side_effect=SQLAlchemyError('test')):
            with self.assertRaises(SQLAlchemyError):
                self.db_ops.executeSADRD_SP(['sp_name'])

    @patch('Services.dboperations.pd.read_sql')
    @patch('Services.dboperations.urllib.parse.quote_plus')
    def test_executeNIR_SP(self, mock_quote_plus, mock_read_sql):
        mock_read_sql.return_value = pd.DataFrame()
        mock_quote_plus.return_value = self.nir_url
        self.db_ops.engine = MagicMock()
        self.db_ops.executeNIR_SP('sp_name', 2023, 'company')

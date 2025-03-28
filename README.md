import unittest
from unittest.mock import patch, Mock
import pandas as pd
from datetime import datetime

# Assuming SADRD_Dataparser.py is in the same directory or in your PYTHONPATH
import SADRD_Dataparser

class TestSADRDDataparser(unittest.TestCase):

    @patch('SADRD_Dataparser.dbops.dboperations')
    @patch('SADRD_Dataparser.gvar')
    def test_parseAnnualStmntFile_success(self, mock_gvar, mock_dbops):
        """Tests successful execution of parseAnnualStmntFile."""

        mock_dbops_instance = Mock()
        mock_dbops.return_value = mock_dbops_instance
        mock_gvar.sadrdYear = 2025  # Example year
        mock_gvar.user_id = "test_user"
        mock_gvar.INPROGRESS = "INPROGRESS"
        mock_gvar.COMPLETED = "COMPLETED"
        mock_gvar.sadrd_settings = [Mock(settingName='SADRD_Year', settingValue='2025')]
        mock_gvar.filesLoadedCount = 0

        mock_dbops_instance.executeNIR_SP.return_value = pd.DataFrame({'Cusip': ['12345-AB', '67890-CD'], 'DividendAmt': [100, 200]})
        mock_dbops_instance.loaddata.return_value = None
        mock_dbops_instance.executeSADRD_SP.return_value = None

        SADRD_Dataparser.parseAnnualStmntFile("TestCompany", "Part 2 Section 1", "TestAction", 2025, "")

        mock_dbops_instance.executeNIR_SP.assert_called_once()
        mock_dbops_instance.insert_dataloadkey.assert_called_once()
        mock_dbops_instance.loaddata.assert_called()  # Called multiple times
        mock_dbops_instance.executeSADRD_SP.assert_called_once()

        self.assertEqual(mock_gvar.filesLoadedCount, 1)

    @patch('SADRD_Dataparser.dbops.dboperations')
    @patch('SADRD_Dataparser.gvar')
    def test_parseAnnualStmntFile_secondaryValidation(self, mock_gvar, mock_dbops):
         """Tests the early return when SecondaryValidation is not empty."""
         mock_dbops_instance = Mock()
         mock_dbops.return_value = mock_dbops_instance
         SADRD_Dataparser.parseAnnualStmntFile("TestCompany", "Part 2 Section 1", "TestAction", 2025, "validation")
         mock_dbops_instance.executeNIR_SP.assert_not_called()

    @patch('SADRD_Dataparser.dbops.dboperations')
    @patch('SADRD_Dataparser.gvar')
    def test_parseAnnualStmntFile_exception(self, mock_gvar, mock_dbops):
        """Tests exception handling in parseAnnualStmntFile."""
        mock_dbops_instance = Mock()
        mock_dbops.return_value = mock_dbops_instance
        mock_dbops_instance.executeNIR_SP.side_effect = Exception("Test Exception")

        with self.assertRaises(Exception) as context:
            SADRD_Dataparser.parseAnnualStmntFile("TestCompany", "Part 2 Section 1", "TestAction", 2025, "")

        self.assertEqual(str(context.exception), "Test Exception")
        mock_dbops_instance.insert_actionLog.assert_called()  # Check that logging occurred

    @patch('SADRD_Dataparser.dbops.dboperations')
    @patch('SADRD_Dataparser.gvar')
    @patch('pandas.ExcelFile')
    def test_parseCusipQualFTCFile_success(self, mock_excel_file, mock_gvar, mock_dbops):
        """Tests successful execution of parseCusipQualFTCFile."""
        mock_dbops_instance = Mock()
        mock_dbops.return_value = mock_dbops_instance
        mock_gvar.sadrdYear = 2025
        mock_gvar.user_id = "test_user"
        mock_gvar.INPROGRESS = "INPROGRESS"
        mock_gvar.COMPLETED = "COMPLETED"
        mock_gvar.fileErrorMessages = ""
        mock_gvar.sadrd_settings = [Mock(settingName='Valid_Company', settingValue='JHFunds'),
                                   Mock(settingName='Valid_Company', settingValue='RPS')]
        mock_gvar.filesLoadedCount = 0

        mock_excel = Mock()
        mock_excel.book.worksheets = [Mock(title='JHFunds', sheet_state='visible')]
        mock_excel.parse.return_value = pd.DataFrame({
            'Year': [2025],
            'Company': ['JHFunds'],
            'Cusip': ['123'],
            'CusipName': ['Test'],
            'Bank': ['Bank'],
            'FTC': [1.0],
            'QualPct': [2.0]
        })
        mock_excel_file.return_value = mock_excel
        mock_dbops_instance.insert_dataloadkey.return_value = None
        mock_dbops_instance.loaddata.return_value = None

        SADRD_Dataparser.parseCusipQualFTCFile("TestAction", "test.xlsx", 2025, "")

        mock_dbops_instance.insert_dataloadkey.assert_called_once()
        mock_dbops_instance.loaddata.assert_called_once()
        self.assertEqual(mock_gvar.filesLoadedCount, 1)

    @patch('SADRD_Dataparser.dbops.dboperations')
    @patch('SADRD_Dataparser.gvar')
    @patch('pandas.ExcelFile')
    def test_parseCusipQualFTCFile_secondaryValidation_year_mismatch(self, mock_excel_file, mock_gvar, mock_dbops):
        """Tests secondary validation with year mismatch."""
        mock_dbops_instance = Mock()
        mock_dbops.return_value = mock_dbops_instance
        mock_gvar.sadrdYear = 2025
        mock_gvar.fileErrorMessages = ""
        mock_gvar.sadrd_settings = [Mock(settingName='Valid_Company', settingValue='JHFunds')]

        mock_excel = Mock()
        mock_excel.book.worksheets = [Mock(title='JHFunds', sheet_state='visible')]
        mock_excel.parse.return_value = pd.DataFrame({'Year': [2024], 'Company': ['JHFunds']}) # Year mismatch
        mock_excel_file.return_value = mock_excel

        SADRD_Dataparser.parseCusipQualFTCFile("TestAction", "test.xlsx", 2025, "validation")

        self.assertIn("E005", mock_gvar.fileErrorMessages)
        mock_dbops_instance.insert_dataloadkey.assert_not_called()

    @patch('SADRD_Dataparser.dbops.dboperations')
    @patch('SADRD_Dataparser.gvar')
    @patch('pandas.ExcelFile')
    def test_parseCusipQualFTCFile_secondaryValidation_company_mismatch(self, mock_excel_file, mock_gvar, mock_dbops):
        """Tests secondary validation with company mismatch."""
        mock_dbops_instance = Mock()
        mock_dbops.return_value = mock_dbops_instance
        mock_gvar.sadrdYear = 2025
        mock_gvar.fileErrorMessages = ""
        mock_gvar.sadrd_settings = [Mock(settingName='Valid_Company', settingValue='JHFunds')]

        mock_excel = Mock()
        mock_excel.book.worksheets = [Mock(title='JHFunds', sheet_state='visible')]
        mock_excel.parse.return_value = pd.DataFrame({'Year': [2025], 'Company': ['WrongCompany']})
        mock_excel_file.return_value = mock_excel

        SADRD_Dataparser.parseCusipQualFTCFile("TestAction", "test.xlsx", 2025, "validation")

        self

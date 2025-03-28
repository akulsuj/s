import json
from datetime import datetime

import pandas as pd , datetime , os

import urllib

from Services import dboperations as dbops
import globalvars as gvar
import logging
import re
import stringcase

dbops_obj = dbops.dboperations()
log = logging.getLogger(__name__)
gvar.sadrd_settings = dbops_obj.SadrdSysSettings()
gvar.sadrd_ErrMessages = dbops_obj.SADRD_Sys_Message()

dataSettingYear = [x for x in gvar.sadrd_settings if x.settingName == 'SADRD_Year']
gvar.sadrdYear = int(dataSettingYear[0].settingValue)

listCompany =[]
'''
dataSettingCompanies = [x for x in gvar.sadrd_settings if x.settingName == 'Valid_Company']
listCompany.append(dataSettingCompanies[0].settingValue)
'''
for (row, sadrdSysSetting) in enumerate(gvar.sadrd_settings):
    if sadrdSysSetting.settingName == "Valid_Company":
        listCompany.append(sadrdSysSetting.settingValue)

def parseAnnualStmntFile(company, partType, actionname, AnnualStmntYear, SecondaryValidation):

    logging.debug("In parseAnnualStmntFile() method")
    try:
        errFileImport = ""     
        file_name = "SchD-NIR Tables"

        loadkey_dict = dict([('companyName', company), ('year', str(gvar.sadrdYear)), ('partType', partType)])

        for index, (key, value) in enumerate(loadkey_dict.items()):
            loadkey = value if index == 0 else loadkey + ";" + value
        
        strJsonVal = json.dumps(loadkey_dict, indent=4)

        # Choosing SP based on PartType
        nirSp = ""
        if partType == "Part 2 Section 1":
            nirSp = "[sadrd].[uspGenerateDP2S1FeedReportToSADRD]"
        elif partType == "Part 2 Section 2":
            nirSp = "[sadrd].[uspGenerateDP2S2FeedReportToSADRD]"
        elif partType == "Part 4":
            nirSp = "[sadrd].[uspGenerateDP4FeedReportToSADRD]"
        elif partType == "Part 5":
            nirSp = "[sadrd].[uspGenerateDP5FeedReportToSADRD]"

        datatb = dbops_obj.executeNIR_SP(nirSp, gvar.sadrdYear, company) 

        if(SecondaryValidation != ""):
            return

        datatb['Cusip'] = datatb['Cusip'].str.replace('-','').fillna('0')
        datatb.insert(8, 'ReportType', '')

        dbops_obj.insert_dataloadkey(loadkey, strJsonVal,  actionname, gvar.INPROGRESS, str(datetime.datetime.now().strftime('%Y-%m-%d'))[0:10], gvar.user_id)
        dbops_obj.insert_actionLog(datetime.datetime.now().month, datetime.datetime.now().year, gvar.user_id, actionname, 'parseAnnualStmntFile()', str(datetime.datetime.now())[0:23], "SADRD_Dataparser - insert_dataloadkey method executed", gvar.dataload_id)

        # Loading dataframe data into control table
        if not datatb.empty:
            datatbTotalDividendAmount = datatb['DividendAmt'].sum()
            ctrldatatb = pd.DataFrame()  
            ctrldatatb['FileName'] = [file_name]
            ctrldatatb['ReportType'] = ['All']
            ctrldatatb['PartType'] = [partType]
            ctrldatatb['Company'] = [company]
            ctrldatatb['Year'] = [gvar.sadrdYear] 
            ctrldatatb['InputFile_Amt'] = [datatbTotalDividendAmount]
            dbops_obj.loaddata('SADRD_FactsTemp_CntrlTotals_Input', ctrldatatb, '')
            dbops_obj.insert_actionLog(datetime.datetime.now().month, datetime.datetime.now().year, gvar.user_id, actionname, 'parseAnnualStmntFile()', str(datetime.datetime.now())[0:23], "SADRD_Dataparser - loaddata(dataframe) method executed", gvar.dataload_id)
        
        dbops_obj.loaddata('SADRD_SrcStaging_SchDAllPartsdata', datatb, '')
        dbops_obj.insert_actionLog(datetime.datetime.now().month, datetime.datetime.now().year, gvar.user_id, actionname, 'parseAnnualStmntFile()', str(datetime.datetime.now())[0:23], "SADRD_Dataparser - loaddata method executed", gvar.dataload_id)
        gvar.filesLoadedCount = gvar.filesLoadedCount + 1

        inputParamsDict = []
        inputParamsDict.append('sp_SADRD_INSERT_SchD_ByLineNo @userName=?')
        inputParamsDict.append(gvar.user_id)
        spOutputResult = dbops_obj.executeSADRD_SP(inputParamsDict)
        dbops_obj.insert_actionLog(datetime.datetime.now().month, datetime.datetime.now().year, gvar.user_id, actionname, 'parseAnnualStmntFile()', str(datetime.datetime.now())[0:23], "SADRD_Dataparser - executeSADRD_SP method executed for SP sp_SADRD_INSERT_SchD_ByLineNo", gvar.dataload_id)
        
    except Exception as e:
        logging.exception("Exception occurred in SADRD_Dataparser - parseAnnualStmntFile() method :: " + str(e)) 
        dbops.logger.error('Error in SADRD_Dataparser - parseAnnualStmntFile() parsefile -' + str(e))
        raise e

def parseCusipQualFTCFile(actionname, inpfile, UIYear, SecondaryValidation):
    logging.debug("In parseCusipQualFTCFile() method")
    try:
        datatb = pd.DataFrame()  
        #errFileImport = ""     
        file_name = os.path.basename(inpfile)
        file_name_1 = file_name.split(".")[0]
        
        if file_name_1 != None and "JHFunds" in file_name_1:
            fileType = 'JHFunds'
        elif file_name_1 != None and "RPS" in file_name_1:
            fileType = 'RPS'

        xl = pd.ExcelFile(inpfile)
        #workbook = xl.open_workbook(inpfile)
        sheets = xl.book.worksheets
        datatb = pd.DataFrame()

        if SecondaryValidation != "":
            errsheetNames=""
            errSheetNameMessage = ""
            for sheet in sheets:
            #for idx, sheetName in enumerate(xl.sheet_names):
                errFileImport = "" 
                sheetName = sheet.title
                if sheet.sheet_state == "visible":               
                    if sheetName not in listCompany:
                        errsheetNames = sheetName if errsheetNames == '' else (errsheetNames + " | " + sheetName)
                    else:

                        sheet = xl.parse(sheetName)
                        sheet = sheet.loc[:, ~sheet.columns.str.contains('Unnamed')]
                    
                        datatb = sheet
                        if datatb.size > 0:
                            dbYear = datatb.Year.unique()
                            yearSize=0
                            yearSize = int(len(dbYear))

                            if yearSize == 1:                
                                distYear = datatb.Year.unique()[0]
                                
                                if int(distYear) != int(gvar.sadrdYear):
                                    errMessage = 'E005' + "," + file_name + "," + sheetName
                                    errFileImport = errMessage if errFileImport == '' else (errFileImport + ";" + errMessage)     
                            elif yearSize > 1: 
                                errMessage = 'E005' + "," + file_name + "," + sheetName
                                errFileImport = errMessage if errFileImport == '' else (errFileImport + ";" + errMessage)     
                            
                            dbCompany = datatb.Company.unique()
                            companySize=0
                            companySize = int(len(dbCompany))

                            # if companySize == 1:
                            #     distCompany = datatb.Company.unique()[0]
                            #     if distCompany not in listCompany:
                            #         errMessage = 'E016' + "," + file_name + "," + sheetName
                            #         errFileImport = errMessage if errFileImport == '' else (errFileImport + ";" + errMessage)     
                            
                            #get list of company should match with sheet name
                            #availability = 'available' if all(i==sheetName for i in dbCompany) else 'booked'
                            intRowCompanyNotMatchingSheetName = 0
                            intRowCompanyNotValid = 0
                            for companyItem in dbCompany:
                                if companyItem != sheetName:
                                    intRowCompanyNotMatchingSheetName = intRowCompanyNotMatchingSheetName + 1
                                if companyItem not in listCompany:
                                    intRowCompanyNotValid = intRowCompanyNotValid + 1
                            if intRowCompanyNotMatchingSheetName !=0: 
                                errMessage = 'E015' + "," + file_name + "," + sheetName
                                errFileImport = errMessage if errFileImport == '' else (errFileImport + ";" + errMessage)     
                            if intRowCompanyNotValid !=0: 
                                errMessage = 'E016' + "," + file_name + "," + sheetName
                                errFileImport = errMessage if errFileImport == '' else (errFileImport + ";" + errMessage)
                        else:
                            errMessage = 'E025' + "," + file_name + "," + sheetName
                            errFileImport = errMessage if errFileImport == '' else (errFileImport + ";" + errMessage)         
                    if errFileImport != "": 
                        gvar.fileErrorMessages = errFileImport if gvar.fileErrorMessages == '' else (gvar.fileErrorMessages + ";" + errFileImport)           
            if errsheetNames!="":
                errSheetNameMessage = 'E020' + "," + file_name + "," + errsheetNames
                gvar.fileErrorMessages = errSheetNameMessage if gvar.fileErrorMessages == '' else (gvar.fileErrorMessages + ";" + errSheetNameMessage)
                    
            return            
            
        else:
            errSheetName =""
            for sheet in sheets:
                sheetName = sheet.title
            #for idx, sheetName in enumerate(xl.sheet_names):
                if sheet.sheet_state == "visible":   
                    if sheetName not in listCompany:
                        errSheetName = sheetName if errSheetName == '' else (errSheetName + "; " + sheetName)                   
                        
                    sheet = xl.parse(sheetName)
                    sheet = sheet.loc[:, ~sheet.columns.str.contains('Unnamed')]
                
                    datatb = sheet
                    distYear = datatb.Year.unique()[0]
                    distCompany = datatb.Company.unique()[0]

                    loadkey_dict = dict(
                            [('companyName', distCompany), ('year', str(distYear)), ('fileType',fileType)])

                    for index, (key, value) in enumerate(loadkey_dict.items()):
                        loadkey = value if index == 0 else loadkey + ";" + value
                    strJsonVal = json.dumps(loadkey_dict, indent=4)

                    datatb = sheet.iloc[0:]
                    datatb.columns =['Year', 'Company', 'Cusip', 'CusipName', 'Bank', 'FTC', 'QualPct']
                    datatb = datatb
                    datatb.insert(7, 'Source', fileType)
                    datatb.insert(8, 'FileName', file_name)

                    datatb['Cusip'] = datatb['Cusip'].astype(str)
                    datatb['CusipName'] = datatb['CusipName'].astype(str)
                    datatb['Bank'] = datatb['Bank'].astype(str)
                    datatb['FTC'] = datatb['FTC'].fillna(0)
                    datatb['QualPct'] = datatb['QualPct'].fillna(0)
                    dbops_obj.insert_dataloadkey(loadkey, strJsonVal,  actionname, gvar.INPROGRESS, str(datetime.datetime.now().strftime('%Y-%m-%d'))[0:10], gvar.user_id)
                    dbops_obj.insert_actionLog(datetime.datetime.now().month, datetime.datetime.now().year, gvar.user_id, actionname, 'parseCusipQualFTCFile', str(datetime.datetime.now())[0:23], "SADRD_Dataparser - insert_dataloadkey method executed", gvar.dataload_id)
                    
                    dbops_obj.loaddata('SADRD_SrcStaging_CusipQualFTC', datatb, '')
                    dbops_obj.insert_actionLog(datetime.datetime.now().month, datetime.datetime.now().year, gvar.user_id, actionname, 'parseCusipQualFTCFile()', str(datetime.datetime.now())[0:23], "SADRD_Dataparser - loaddata method executed for SADRD_SrcStaging_CusipQualFTC", gvar.dataload_id)
                    

                    if errSheetName !="":
                        errSheetMessage = 'E020' + "," + file_name + "," + errSheetName
                        errMessageComplete = dbops_obj.BuildErrorMessage(str(errSheetMessage))
                        dbops_obj.insert_actionLog(datetime.datetime.now().month, datetime.datetime.now().year, gvar.user_id, actionname, 'parseCusipQualFTCFile()', str(datetime.datetime.now())[0:23], errMessageComplete, gvar.dataload_id)
            gvar.filesLoadedCount = gvar.filesLoadedCount + 1                   
    except Exception as e:
        logging.exception("Exception occurred in SADRD_Dataparser - parseCusipQualFTCFile() method :: " + str(e)) 
        dbops_obj.insert_actionLog(datetime.datetime.now().month, datetime.datetime.now().year, gvar.user_id, actionname, 'parseCusipQualFTCFile()', str(datetime.datetime.now())[0:23], ("Exception occurred in SADRD_Dataparser - parseCusipQualFTCFile() - " + str(e)), None)
        dbops.logger.error('Error in SADRD_Dataparser - parseCusipQualFTCFile() parsefile -' + str(e))
        raise e

def parseJHFundsFTCGrossupFile(actionname, inpfile, FTCGrossupYear, SecondaryValidation):
    logging.debug("In parseJHFundsFTCGrossupFile() method")
    try:
        datatb = pd.DataFrame()  
        errFileImport = ""     
        file_name = os.path.basename(inpfile)
        file_name_1 = file_name.split(".")[0]
        
        fileComponents = file_name_1.split("_")
        fileNameYear = fileComponents[0]
        fileQuarter = fileComponents[1]
        fileType = fileComponents[2]
        xl = pd.ExcelFile(inpfile)
        #workbook = xl.open_workbook(inpfile)
        sheets = xl.book.worksheets
        datatb = pd.DataFrame()

        if SecondaryValidation !="":
            
            for sheet in sheets:
            #for idx, sheetName in enumerate(xl.sheet_names):
                errFileImport = "" 
                sheetName = sheet.title
                if sheet.sheet_state == "visible": 
                    sheet = xl.parse(sheetName)
                    sheet = sheet.loc[:, ~sheet.columns.str.contains('Unnamed')]
                
                    datatb = sheet
                    if datatb.size > 0:
                        datatb = datatb[datatb['Year'].notna()]
                        datatb['Year'] = datatb['Year'].astype(int)
                        datatb['Quarter'] = datatb['Quarter'].astype(int)
                        dbYear = datatb.Year.unique()
                        yearSize=0
                        yearSize = int(len(dbYear))

                        if yearSize == 1:                
                            distYear = datatb.Year.unique()[0]
                            
                            if int(distYear) != int(gvar.sadrdYear):
                                errMessage = 'E005' + "," + file_name + "," + sheetName
                                errFileImport = errMessage if errFileImport == '' else (errFileImport + ";" + errMessage)     
                        elif yearSize > 1: 
                            errMessage = 'E005' + "," + file_name + "," + sheetName
                            errFileImport = errMessage if errFileImport == '' else (errFileImport + ";" + errMessage)     

                        distQuarter = datatb.Quarter.unique()
                        quarterSize = 0     
                        quarterSize = int(len(distQuarter))

                        if quarterSize == 1: 
                            distQuarter = datatb.Quarter.unique()[0]
                            
                            if int(distQuarter) != int(fileQuarter[1]):
                                errMessage = 'E010' + "," + file_name + "," + sheetName
                                errFileImport = errMessage if errFileImport == '' else (errFileImport + ";" + errMessage)     
                        elif quarterSize > 1: 
                            errMessage = 'E010' + "," + file_name + "," + sheetName
                            errFileImport = errMessage if errFileImport == '' else (errFileImport + ";" + errMessage)     
           
            if errFileImport != "":
                gvar.fileErrorMessages = errFileImport if gvar.fileErrorMessages == '' else (gvar.fileErrorMessages + ";" + errFileImport)
            return

        else:
            errSheetName =""
            for sheet in sheets:
                sheetName = sheet.title
            #for idx, sheetName in enumerate(xl.sheet_names):
                if sheet.sheet_state == "visible":   
                    
                    sheet = xl.parse(sheetName)
                    sheet = sheet.loc[:, ~sheet.columns.str.contains('Unnamed')]
                
                    #datatb = sheet
                    datatb = sheet.iloc[0:]
                    datatb = datatb[datatb['Year'].notna()]
                    datatb['Year'] = datatb['Year'].astype(int)
                    datatb['Quarter'] = datatb['Quarter'].astype(int)
                    distYear = datatb.Year.unique()[0]
                    
                    loadkey_dict = dict([('year', fileNameYear), ('fileQuarter', fileQuarter), ('fileType', fileType)])

                    for index, (key, value) in enumerate(loadkey_dict.items()):
                        loadkey = value if index == 0 else loadkey + ";" + value
                    
                    strJsonVal = json.dumps(loadkey_dict, indent=4)

                    


                    #datatb = datatb[['Year', 'Quarter', 'Bank', 'JHUSA%Share','JHNY%Share','Other%Share']]
                    #datatb = datatb.rename({'JHUSA%Share': 'JHUSAPctShare', 'JHNY%Share': 'JHNYPctShare', 'Other%Share': 'OtherPctShare'}, axis=1)  
        

                    #datatb.columns =['Year', 'Bank', 'JHUSA%Share', 'JHNY%Share', 'Bank', 'FTC', 'QualPct']
                    
                    datatb = datatb[['Year', 'Quarter', 'Bank', 'Trust', 'JHNY%Share', 'JHUSA%Share','Other%Share', 'Total%Share', 'Fund Name', 'JHNY-Div', 'JHUSA-Div','Other-Div', 'Total All-Div']]
                    datatb = datatb.rename({'JHUSA%Share': 'JHUSAPctShare', 'JHNY%Share': 'JHNYPctShare', 'Other%Share': 'OtherPctShare', 'Total%Share': 'TotalPctShare', 'Fund Name' : 'FundName', 'JHNY-Div' : 'JHNYDividend', 'JHUSA-Div' : 'JHUSADividend', 'Other-Div' : 'OtherDividend', 'Total All-Div' : 'TotalDividend'}, axis=1)  
                    #datatb = datatb
                    datatb.insert(13, 'FileName', file_name)

                    datatb['Bank'] = datatb['Bank'].astype(str)                    
                    datatb['JHUSAPctShare'] = datatb['JHUSAPctShare'].fillna(0)
                    datatb['JHNYPctShare'] = datatb['JHNYPctShare'].fillna(0)
                    datatb['OtherPctShare'] = datatb['OtherPctShare'].fillna(0)
                    datatb['TotalPctShare'] = datatb['TotalPctShare'].fillna(0)
                    datatb['JHNYDividend'] = datatb['JHNYDividend'].fillna(0)
                    datatb['JHUSADividend'] = datatb['JHUSADividend'].fillna(0)
                    datatb['OtherDividend'] = datatb['OtherDividend'].fillna(0)
                    datatb['TotalDividend'] = datatb['TotalDividend'].fillna(0)
                    datatb['FileName'] = datatb['FileName'].astype(str)      
                    dbops_obj.insert_dataloadkey(loadkey, strJsonVal,  actionname, gvar.INPROGRESS, str(datetime.datetime.now().strftime('%Y-%m-%d'))[0:10], gvar.user_id)
                    dbops_obj.insert_actionLog(datetime.datetime.now().month, datetime.datetime.now().year, gvar.user_id, actionname, 'parseJHFundsFTCGrossupFile()', str(datetime.datetime.now())[0:23], "insert_dataloadkey method executed", gvar.dataload_id)
                    
                    dbops_obj.loaddata('SADRD_SrcStaging_BankGrossUpPct', datatb, '')
                    dbops_obj.insert_actionLog(datetime.datetime.now().month, datetime.datetime.now().year, gvar.user_id, actionname, 'parseJHFundsFTCGrossupFile()', str(datetime.datetime.now())[0:23], "SADRD_Dataparser - loaddata method executed for SADRD_SrcStaging_BankGrossUpPct", gvar.dataload_id)
                    
                    gvar.filesLoadedCount = gvar.filesLoadedCount + 1

                    inputParamsDict = []
                    
                    inputParamsDict.append('sp_SADRD_INSERT_BankGrossUpPct @fileName=?, @updateControlTotalsFlag=?, @ReturnCode=?')
                    inputParamsDict.append(file_name)
                    inputParamsDict.append('N')
                    inputParamsDict.append('Y')
                    spOutputResult = dbops_obj.executeSADRD_SP(inputParamsDict)
                    if spOutputResult != 'Y' and 'E026' not in gvar.fileErrorMessages:
                        errMessage = 'E026' + "," + file_name + "," + sheetName
                        errFileImport = errMessage if errFileImport == '' else (errFileImport + ";" + errMessage)     
                        
                    dbops_obj.insert_actionLog(datetime.datetime.now().month, datetime.datetime.now().year, gvar.user_id, actionname, 'parseJHFundsFTCGrossupFile()', str(datetime.datetime.now())[0:23], "SADRD_Dataparser - executeSADRD_SP method executed for SP sp_SADRD_INSERT_BankGrossUpPct", gvar.dataload_id)
            if errFileImport != "":
                gvar.fileErrorMessages = errFileImport if gvar.fileErrorMessages == '' else (gvar.fileErrorMessages + ";" + errFileImport)
            return

    except Exception as e:
        logging.exception("Exception occurred in SADRD_Dataparser - parseJHFundsFTCGrossupFile() method :: " + str(e)) 
        dbops_obj.insert_actionLog(datetime.datetime.now().month, datetime.datetime.now().year, gvar.user_id, actionname, 'parseJHFundsFTCGrossupFile()', str(datetime.datetime.now())[0:23], ("Exception occurred in SADRD_Dataparser - parseJHFundsFTCGrossupFile() - " + str(e)), None)
        dbops.logger.error('Error in SADRD_Dataparser - parsefile -' + str(e))
        raise e

def executeView_GetDetails(importType, cusipType):
    logging.debug("In executeView_GetDetails() method")
    try:
        viewDetails = dbops_obj.executeView_GetDetails(importType, cusipType)
        dbops_obj.insert_actionLog(datetime.datetime.now().month, datetime.datetime.now().year, gvar.user_id, importType, 'executeView_GetDetails()', str(datetime.datetime.now())[0:23], "SADRD_Dataparser - executeView_GetDetails method executed", None)
    except Exception as e:
        logging.exception("Exception occurred in SADRD_Dataparser - executeView_GetDetails() method :: " + str(e)) 
        if "generateSADRDReport" == importType:
            dbops.error('Exception occured in executing view SADRD_Dataparser - vSADRD_UVCusips or vSADRD_VPACusips: ' + str(e))
        raise e
    return viewDetails

def Load_FundCusipDetails(actionname, dfCusipDetails, fundCusipYear):
    logging.debug("In Load_FundCusipDetails() method")
    try:
        dbops_obj.insert_dataloadkey('Loading data to SADRD_Srctemp_CusipMapping table'+';'+actionname, '', actionname, gvar.COMPLETED, str(datetime.datetime.now().strftime('%Y-%m-%d'))[0:10],gvar.user_id)
        dbops_obj.insert_actionLog(datetime.datetime.now().month, datetime.datetime.now().year, gvar.user_id, actionname, 'Load_FundCusipDetails()', str(datetime.datetime.now())[0:23], "SADRD_Dataparser - Load_FundCusipDetails-Loading data to SADRD_Srctemp_CusipMapping table started", gvar.dataload_id)
        dfCusipDetails.insert(0, 'Year', fundCusipYear)
        dbops_obj.loaddata('SADRD_Srctemp_CusipMapping' , dfCusipDetails, '')
        dbops_obj.insert_actionLog(datetime.datetime.now().month, datetime.datetime.now().year, gvar.user_id, actionname, 'Load_FundCusipDetails()', str(datetime.datetime.now())[0:23], "SADRD_Dataparser - Load_FundCusipDetails-Loading data to SADRD_Srctemp_CusipMapping table completed", gvar.dataload_id)
    except Exception as e:
        logging.exception("Exception occurred in SADRD_Dataparser - Load_FundCusipDetails() method :: " + str(e)) 
        dbops_obj.insert_actionLog(datetime.datetime.now().month, datetime.datetime.now().year, gvar.user_id, actionname, 'Load_FundCusipDetails', str(datetime.datetime.now())[0:23], ("Exception occurred in SADRD_Dataparser - Load_FundCusipDetails() - " + str(e)), None)
        dbops.error('Error occured in method SADRD_Dataparser - Load_FundCusipDetails() while loading data using SP SADRD_Srctemp_CusipMapping: ' + str(e))
        raise e

def LoadVPADataToSADRD(actionName, fundCusipYear, dataType):
    logging.debug("In LoadVPADataToSADRD() method")
    try:
        vpaData = dbops_obj.GetVPAData(actionName, fundCusipYear, dataType, '')
        if dataType == 'GL':
            dbops_obj.truncatetable('SADRD_FactsTemp_CntrlTotals_Input', "GL")
            gvar.sqlconfig = urllib.parse.quote_plus("DRIVER=" + gvar.gconfig["DRIVER"] + ";" + "SERVER=" + gvar.gconfig["VPA_DATABASE_SERVER"] + ";" + "DATABASE=" + gvar.gconfig["VPA_DATABASE_NAME"] + gvar.gconfig["VPA_DATABASE_AUTH_STRING"])
            vpaControlTotalsData = dbops_obj.GetVPAData(actionName, fundCusipYear, dataType, 'CtrlTot')
            for i in range(len(vpaControlTotalsData)):
                dbops_obj.InsertControlTotals(actionName, vpaControlTotalsData.loc[i, 'Year'], vpaControlTotalsData.loc[i, 'Company'], vpaControlTotalsData.loc[i, 'Amount'])
            dbops_obj.insert_dataloadkey('Loading current year data from VPA GL table to SADRD GL table' + ';' + actionName, '', actionName, gvar.COMPLETED, str(datetime.datetime.now().strftime('%Y-%m-%d'))[0:10], gvar.user_id)
            dbops_obj.insert_actionLog(datetime.datetime.now().month, datetime.datetime.now().year, gvar.user_id, actionName, 'LoadVPADataToSADRD()-VPA-GL', str(datetime.datetime.now())[0:23], "SADRD_Dataparser - LoadVPADataToSADRD-VPA GL table to SADRD GL table - started", gvar.dataload_id)
            dbops_obj.loaddata('SADRD_GL', vpaData, '')
            dbops_obj.insert_actionLog(datetime.datetime.now().month, datetime.datetime.now().year, gvar.user_id, actionName, 'LoadVPADataToSADRD()-VPA-GL', str(datetime.datetime.now())[0:23], "SADRD_Dataparser - LoadVPADataToSADRD-VPA GL table to SADRD GL table - completed", gvar.dataload_id)
        elif dataType == 'SeparateAccounts':
            dbops_obj.insert_dataloadkey('Loading data from VPA SeparateAccounts table to SADRD SeparateAccounts table' + ';' + actionName, '', actionName, gvar.COMPLETED, str(datetime.datetime.now().strftime('%Y-%m-%d'))[0:10], gvar.user_id)
            dbops_obj.insert_actionLog(datetime.datetime.now().month, datetime.datetime.now().year, gvar.user_id, actionName, 'LoadVPADataToSADRD()-VPA-SeparateAccounts', str(datetime.datetime.now())[0:23], "SADRD_Dataparser - LoadVPADataToSADRD-VPA SeparateAccounts table to SADRD SeparateAccounts table - started", gvar.dataload_id)
            dbops_obj.loaddata('SADRD_SeparateAccounts', vpaData, '')
            dbops_obj.insert_actionLog(datetime.datetime.now().month, datetime.datetime.now().year, gvar.user_id, actionName, 'LoadVPADataToSADRD()-VPA-SeparateAccounts', str(datetime.datetime.now())[0:23], "SADRD_Dataparser - LoadVPADataToSADRD-VPA SeparateAccounts table to SADRD SeparateAccounts table - completed", gvar.dataload_id)
    except Exception as e:
        logging.exception("Exception occurred in SADRD_Dataparser - LoadVPADataToSADRD() method :: " + str(e)) 
        dbops_obj.insert_actionLog(datetime.datetime.now().month, datetime.datetime.now().year, gvar.user_id, actionName, 'LoadVPADataToSADRD', str(datetime.datetime.now())[0:23], ("Exception occurred in SADRD_Dataparser - LoadVPADataToSADRD() - " + str(e)), None)
        dbops.error('Exception occured in SADRD_Dataparser - LoadVPADataToSADRD while loading data from VPA GL to SADRD GL :: ' + str(e))
        raise e

| MEASURE_NAME                     | EXPRESSION |
|---------------------------------|------------|
| Sales                             | `SUMX('Sales Files',[QTY_Case]+([Qty_Bottle]/[Case Size]))` |
| Sales YTD                         | `IF([ShowValueForDates], CALCULATE([Sales], DATESYTD(Calendar[Date])))` |
| Sum of QTY_Case                   | `SUM('Sales Files'[QTY_Case])` |
| Sum of Qty_Bottle                  | `SUM('Sales Files'[Qty_Bottle])` |
| Category Sales                     | `CALCULATE([Sales], ALLEXCEPT('ProductMaster', ProductMaster[Category]))` |
| MS %                               | `IFERROR([Sales]/[Category Sales], BLANK())` |
| Sales MTD                          | `IF([ShowValueForDates], CALCULATE([Sales], DATESMTD(Calendar[Date])))` |
| Category Sales YTD                 | `IF([ShowValueForDates], CALCULATE([Category Sales], DATESYTD(Calendar[Date])))` |
| Category Sales MTD                 | `IF([ShowValueForDates], CALCULATE([Category Sales], DATESMTD(Calendar[Date])))` |
| YTD MS %                           | `IFERROR([Sales YTD]/[Category Sales YTD], BLANK())` |
| MTD MS %                           | `IFERROR([Sales MTD]/[Category Sales MTD], BLANK())` |
| Unique Customers Billed            | `CALCULATE(DISTINCTCOUNT(StoreMaster[Store Code]), FILTER('Sales Files',[Sales]>0))` |
| Sales PYTD                         | ```IF ( [ShowValueForDates], CALCULATE ( [Sales], CALCULATETABLE ( DATEADD('Calendar'[Date], -1, YEAR), 'Calendar'[DateWithSales] = TRUE ) ) )``` |
| Growth % YTD vs PYTD               | ```VAR ValueCurrentPeriod = [Sales] VAR ValuePreviousPeriod = [Sales PYTD] VAR Delta = IF(NOT ISBLANK(ValueCurrentPeriod) && NOT ISBLANK(ValuePreviousPeriod), ValueCurrentPeriod - ValuePreviousPeriod) RETURN DIVIDE(Delta,[Sales PYTD])``` |
| YTD Growth Arrow                   | `SWITCH(TRUE(), [Growth % YTD vs PYTD] <= 0, UNICHAR(9660), [Growth % YTD vs PYTD] >= 0.0001, UNICHAR(9650), BLANK())` |
| Sales PYC                          | ```IF([ShowValueForDates] && HASONEVALUE('Calendar'[Year]), CALCULATE([Sales], PREVIOUSYEAR('Calendar'[Date])))``` |
| Category Sales PYC                 | ```IF([ShowValueForDates] && HASONEVALUE('Calendar'[Year]), CALCULATE([Category Sales], PREVIOUSYEAR('Calendar'[Date])))``` |
| MS % PYC                           | `DIVIDE([Sales PYC],[Category Sales PYC])` |
| My Company MS %                    | `CALCULATE([MS %], ProductMaster[Company]="My Company")` |
| My Company YTD MS %                | `CALCULATE([YTD MS %], ProductMaster[Company]="My Company")` |
| My Company PYC MS %                | `CALCULATE([MS % PYC], ProductMaster[Company]="My Company")` |
| Delta MS % YTD vs PYC              | `[My Company YTD MS %] - [My Company PYC MS %]` |
| Delta MS % YTD vs PYC Arrow        | `SWITCH(TRUE(), [Delta MS % YTD vs PYC] <= 0, UNICHAR(9660), [Delta MS % YTD vs PYC] >= 0.0001, UNICHAR(9650), BLANK())` |
| My Company MTD MS %                | `CALCULATE([MTD MS %], ProductMaster[Company]="My Company")` |
| Sales PMTD                         | ```IF([ShowValueForDates], CALCULATE([Sales], CALCULATETABLE(DATEADD(Calendar[Date], -1, MONTH), Calendar[DateWithSales] = TRUE)))``` |
| Category Sales PMTD                | ```IF([ShowValueForDates], CALCULATE([Category Sales], CALCULATETABLE(DATEADD(Calendar[Date], -1, MONTH), Calendar[DateWithSales] = TRUE)))``` |
| MS % PMTD                          | `IFERROR([Sales PMTD]/[Category Sales PMTD], BLANK())` |
| My Company PMTD MS %               | `CALCULATE([MS % PMTD], ProductMaster[Company]="My Company")` |
| Delta MS % MTD vs PMTD             | `[My Company MTD MS %] - [My Company PMTD MS %]` |
| Delta MS % MTD vs PMTD Arrow       | `SWITCH(TRUE(), [Delta MS % MTD vs PMTD] <= 0, UNICHAR(9660), [Delta MS % MTD vs PMTD] >= 0.0001, UNICHAR(9650), BLANK())` |
| Growth % MTD vs PMTD               | ```VAR ValueCurrentPeriod = [Sales MTD] VAR ValuePreviousPeriod = [Sales PMTD] VAR Delta = IF(NOT ISBLANK(ValueCurrentPeriod) && NOT ISBLANK(ValuePreviousPeriod), ValueCurrentPeriod - ValuePreviousPeriod) RETURN DIVIDE(Delta,[Sales PYTD])``` |
| MTD Growth Arrow                   | `SWITCH(TRUE(), [Growth % MTD vs PMTD] <= 0, UNICHAR(9660), [Growth % MTD vs PMTD] >= 0.0001, UNICHAR(9650), BLANK())` |
| Total Sales                        | `CALCULATE([Sales], ALL(StoreMaster[STATE]))` |
| Contribution Of Total              | `IF([Sales]>0, [Sales]/[Total Sales])` |
| New Customers                       | ```VAR currentDate = MIN('Sales Files'[Date]) VAR currentCustomers = CALCULATETABLE(VALUES('Sales Files'[Store Code]), 'Sales Files'[Date] >= currentDate) VAR pastCustomers = CALCULATETABLE(VALUES('Sales Files'[Store Code]), ALL('Calendar'[Date]), 'Calendar'[Date] < currentDate) VAR newCustomers = EXCEPT(currentCustomers, pastCustomers) RETURN COUNTROWS(DISTINCT(newCustomers))``` |
| Repeat Customers                    | `IFERROR([Unique Customers Billed]-[New Customers], "")` |
| Date of Last Purchase               | `LASTDATE('Sales Files'[Date])` |
| Days Since Last Purchase            | `VALUE(CALCULATE(MAX('Sales Files'[Date]), ALL('Sales Files')) - [Date of Last Purchase])` |
| Lost Customers                      | `CALCULATE(DISTINCTCOUNT('Sales Files'[Store Code]), FILTER('Sales Files', [Days Since Last Purchase] > 30))` |
| Sum of Sort Order                    | `SUM('PackSizeMaster'[Sort Order])` |
| ShowValueForDates                   | ```VAR LastDateWithData = CALCULATE(MAX('Sales Files'[Date]), ALL('Sales Files')) VAR FirstDateVisible = MIN('Calendar'[Date]) VAR Result = FirstDateVisible <= LastDateWithData RETURN Result``` |

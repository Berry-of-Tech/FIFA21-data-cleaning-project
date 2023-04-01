# FIFA21-data-cleaning-project
---
### 1. **Introduction**

Being a newbie in the data space, I decided to participate in this challenge after I completed my SQL course on UDACITY. The challenge was organized by the Twitter data community which gives newbies the opportunity to create a portfolio project. As a participant I used SQL server as a tool. Let me introduce the dataset. The dataset was gotten from Kaggle.

![](https://github.com/Berry-of-Tech/FIFA21-data-cleaning-project/blob/main/Fifa%2021.jpg)

[FIFA 21 messy, raw dataset for cleaning/exploring | Kaggle](https://www.kaggle.com/datasets/yagunnersya/fifa-21-messy-raw-dataset-for-cleaning-exploring)

---

### 2. **What to consider when cleaning your data**

- Incorrect data types
- Null entries
- Duplicate rows/columns
- Misspelling
- Missing values
- Irrelevant data
- Outliers

---

### 3. **Preview of the unclean data**

![](https://github.com/Berry-of-Tech/FIFA21-data-cleaning-project/blob/main/Unclean%20data%201.png)

![](https://github.com/Berry-of-Tech/FIFA21-data-cleaning-project/blob/main/Unclean%20data%202.png)

![](https://github.com/Berry-of-Tech/FIFA21-data-cleaning-project/blob/main/Unclean%20data%203.png)

---

### 4. **Observations from our previewed data**

- Number of rows = 18979
- Number of columns = 77
- SM, IR and W/F columns has special characters which has to be removed.
- Height and Weight columns are not consistent, in the height column, some rows are represented in cm while some are represented in feet/inches. In the weight column some rows are represented in kg while others are represented in lbs.
- The value, Wage and Release clause columns are represented in M and K which are the million and the thousand sign respectively and as well as the euro sign starting each value.
- The loan date end column has numerous amount of null values.
- The hits column have rows that are represented by K in abbreviation of thousand and also contain null values.
- The club columns have some rows with unwanted characters like '1' and '.'
- Misspelling in the LongName column

---

### 5. **The data-cleaning process**

I. We remove the '★' sign in the IR, SM, and W/F columns and convert them to the right data type.

```SQL
-- To remove the star character in IR, [W F] & SM columns
UPDATE fifa_data
SET IR = REPLACE(IR, N'★', N'')
WHERE CHARINDEX(N'★', IR) >0

UPDATE fifa_data
SET [W F] = REPLACE([W F], N'★', N'')
WHERE CHARINDEX(N'★', [W F]) >0

UPDATE fifa_data
SET SM = REPLACE(SM, N'★', N'')
WHERE CHARINDEX(N'★', SM) >0

-- To change the IR, SM AND [W F] data type
ALTER TABLE fifa_data
ALTER COLUMN IR INT

ALTER TABLE fifa_data
ALTER COLUMN [W F] INT

ALTER TABLE fifa_data
ALTER COLUMN SM INT
```

Result after cleaning

![](https://github.com/Berry-of-Tech/FIFA21-data-cleaning-project/blob/main/clean%20SM%20and%20IR%20data.png)

---

II. Updating the height and weight columns for consistency.
```SQL
-- converting all values in height to cm
UPDATE fifa_data
SET Height = CASE  WHEN height LIKE '%''%"' THEN 
            TRY_CONVERT(DECIMAL(10,0), SUBSTRING(height, 1, CHARINDEX('''', height)-1))*30.48 + 
            TRY_CONVERT(DECIMAL(10,0), SUBSTRING(height, CHARINDEX('''', height)+1, LEN(height)-CHARINDEX('''', height)-1))*2.54 
        WHEN height LIKE '%"' THEN TRY_CONVERT(DECIMAL(10,0), SUBSTRING(height, 1, LEN(height) - 2)) * 2.54 
        ELSE TRY_CONVERT(DECIMAL(10,0), SUBSTRING(height, 1, LEN(height) - 2)) 
    END

-- converting all values in weight to lbs
UPDATE fifa_data
SET weight = CASE  WHEN weight like '%kg' THEN TRY_CONVERT(DECIMAL(18,0), SUBSTRING(weight,1, LEN(weight) - 2)) * 2.20462 
				ELSE TRY_CONVERT(DECIMAL(18,0), SUBSTRING(weight,1, LEN(weight) - 3)) END
```

The result after the cleaning

![](https://github.com/Berry-of-Tech/FIFA21-data-cleaning-project/blob/main/clean%20height%20and%20weight%20data.png)

---

III. Next we clean the value, wage, and release clause columns by converting them to appropriate values and removing the euro sign.

```SQL
-- Converting value column to whole figure and removing the euro sign
UPDATE fifa_data
SET Value = CASE WHEN Value LIKE '€%' AND Value LIKE '%M' THEN TRY_CONVERT(DECIMAL(10,2), REPLACE(REPLACE(Value,'€', ''),'M','')) * 1000000
						WHEN Value LIKE '€%' AND Value LIKE '%K' THEN TRY_CONVERT(DECIMAL(10,2), REPLACE(REPLACE(Value,'€', ''),'K','')) * 1000
							ELSE Value END							
WHERE Value like '€%' AND Value like '%M' OR value like '%K'

UPDATE fifa_data
SET value = REPLACE(value, '€', '')

-- Converting wage column to whole figure and removing the euro sign
UPDATE fifa_data
SET Wage = CASE WHEN Wage LIKE '€%' AND Wage LIKE '%M' THEN TRY_CONVERT(DECIMAL(10,2), REPLACE(REPLACE(Wage,'€', ''),'M','')) * 1000000
						WHEN Wage LIKE '€%' AND Wage LIKE '%K' THEN TRY_CONVERT(DECIMAL(10,2), REPLACE(REPLACE(Wage,'€', ''),'K','')) * 1000
							ELSE Wage END							
WHERE Wage like '€%' AND Wage like '%M' OR Wage like '%K'

UPDATE fifa_data
SET Wage = REPLACE(Wage, '€', '')

-- Converting realease clause column to whole figure and removing the euro sign
UPDATE fifa_data
SET [Release Clause] = CASE WHEN [Release Clause] LIKE '€%' AND [Release Clause] LIKE '%M' THEN TRY_CONVERT(DECIMAL(10,2), REPLACE(REPLACE([Release Clause],'€', ''),'M','')) * 1000000
						WHEN [Release Clause] LIKE '€%' AND [Release Clause] LIKE '%K' THEN TRY_CONVERT(DECIMAL(10,2), REPLACE(REPLACE([Release Clause],'€', ''),'K','')) * 1000
							ELSE [Release Clause] END							
WHERE [Release Clause] like '€%' AND [Release Clause] like '%M' OR [Release Clause] like '%K'

UPDATE fifa_data
SET [Release Clause] = REPLACE([Release Clause], '€', '')

```
The result after the cleaning

![](https://github.com/Berry-of-Tech/FIFA21-data-cleaning-project/blob/main/clean%20value%20and%20wage%20.png)

---

IV. Next we update the contract column delimiter and create two new columns, the contract start year and the contract end year and populate it using the contract column.

```SQL
-- Updating contract column delimiter
UPDATE fifa_data
SET contract = REPLACE(contract, '~', '-')
WHERE contract LIKE '%~%'


-- Updating contract column written in text 'on loan'
UPDATE fifa_data
SET contract = 'on loan'
WHERE contract LIKE '%On Loan%'


-- To split the contract column into two different columns
-- We created two new columns called contract_start_year and contract_end_year

ALTER TABLE fifa_data
ADD contract_start_year VARCHAR(25), contract_end_year VARCHAR(25)

-- Populate the new columns using contract column
UPDATE fifa_data
SET contract_start_year = CASE 
							WHEN contract LIKE '%Free%' THEN 'Free'
							WHEN contract LIKE '%loan%' THEN 'On loan'
							ELSE CAST(YEAR(CAST(LEFT(contract,4) +'-01-01' AS date)) AS VARCHAR(25))
							END,
    contract_end_year = CASE 
							WHEN contract LIKE '%Free%' THEN 'Free'
							WHEN contract LIKE '%loan%' THEN 'On loan'
							ELSE CAST(YEAR(CAST(RIGHT(contract,4) +'-01-01' AS date)) AS VARCHAR(25))
							END
WHERE contract LIKE '%-%' OR contract LIKE '%Free%' OR contract LIKE '%loan%'
```
The result after cleaning
![](https://github.com/Berry-of-Tech/FIFA21-data-cleaning-project/blob/main/clean%20split%20contract.png)

---

V. Next we remove unwanted characters from the club and hits columns and update the hits column by converting K and adding 0 to the nulls value for easy analysis.

```SQL
-- Removing irrelevant characters such as 1 and . in club column
UPDATE fifa_data
SET club = REPLACE(REPLACE(club, '.',''), '1', '')


-- Converting the K in hits to round figure
UPDATE fifa_data
SET hits = CASE WHEN UPPER(hits) LIKE '%K' 
   THEN TRY_CONVERT(DECIMAL(18,0), SUBSTRING(hits,1,Len(hits) -1)) * 1000
   ELSE TRY_CONVERT(DECIMAL(18,0), hits) 
   END

-- To replace NULL in hits to 0
UPDATE fifa_data
SET hits =  CASE WHEN hits IS NULL
			 THEN 0
			ELSE hits END
```

The result after cleaning

![](https://github.com/Berry-of-Tech/FIFA21-data-cleaning-project/blob/main/clean%20club%20and%20hits%20data.png)

---

VI. Next, we used the playerURL column to get the clean full name of the players and update the LongName column and also check for duplicate

```SQL
-- Create a new column called player name
ALTER TABLE fifa_data
ADD player_name VARCHAR(100)

-- Populate player name with the player url column
UPDATE fifa_data
SET player_name = SUBSTRING(playerurl, 33, LEN(Longname)+2)

-- To update the Longname column with the player_name column
UPDATE fifa_data
SET LongName = REPLACE(UPPER(
				CASE WHEN CHARINDEX('/', player_name) > 0 
				THEN SUBSTRING(player_name, 1, CHARINDEX('/', player_name)-1)
				ELSE player_name END
				), '-',' ')


-- To remove the extra spaces in the longname column
UPDATE fifa_data
SET LongName = TRIM(LongName)
```

The result after cleaning

![](https://github.com/Berry-of-Tech/FIFA21-data-cleaning-project/blob/main/clean%20fullname%20data%20.png)

---

VII. To check if there are duplicate

```SQL
--To check if there are duplicates
SELECT ID, COUNT(*) duplicate_num
FROM fifa_data
GROUP BY ID
HAVING COUNT(*) > 1
```
The result after checking

![](https://github.com/Berry-of-Tech/FIFA21-data-cleaning-project/blob/main/clean%20duplicate%20ID.png)

There are no duplicates, we used the ID column to check for duplicates because it's a unique column which must not have duplicates, however there are duplicates in the name column, this shows that they have different contracts in different years.

---

VIII. Changing column names, all columns to the right data type and dropping irrelevant columns.

i. PhotoURL and Player URL: This is the link to the players photo which will not add any insight during analysis

ii. Loan Date End: This column contains lots of null values making the loan date end column irrelevant.
```SQL
 --Change column name, drop unwanted columns and set columns to their right data type 

 -- To drop the contract column
 ALTER TABLE fifa_data
 DROP COLUMN contract

 -- To drop the PhotoUrl and playerUrl columns since it's irrelevant
 ALTER TABLE fifa_data
 DROP COLUMN photoUrl

 ALTER TABLE fifa_data
 DROP COLUMN playerUrl


 -- To drop the player name column
 ALTER TABLE fifa_data
 DROP COLUMN player_name


 -- to create another column for the value, wage and release clause to change their data type
 ALTER TABLE fifa_data
 ADD [value(€)] MONEY
 
  ALTER TABLE fifa_data
 ADD [wage(€)] MONEY
 
  ALTER TABLE fifa_data
 ADD [release_clause(€)] MONEY
 
 -- To update the new columns
 UPDATE fifa_data
 SET [value(€)] = CONVERT(money, value )
 
 UPDATE fifa_data
 SET [wage(€)] = CONVERT(money, wage)

  UPDATE fifa_data
 SET [release_clause(€)] = CONVERT(money, [Release clause] )
 


 -- To drop all old value, wage and release clause columns
 ALTER TABLE fifa_data
 DROP COLUMN value
 
  ALTER TABLE fifa_data
DROP COLUMN wage
 
  ALTER TABLE fifa_data
 DROP COLUMN [Release Clause]


  -- To drop the loan date end column since most of the values are NULL
    ALTER TABLE fifa_data
  DROP COLUMN [Loan Date END]

  -- Renaming columns for easy understanding
EXEC sp_rename 'fifa_data.IR', 'IR(injury_resistance)', 'COLUMN'

EXEC sp_rename 'fifa_data.W F', 'W F(weaker_foot_ability)', 'COLUMN'

EXEC sp_rename 'fifa_data.SM', 'SM(skill_moves_ability)', 'COLUMN'

EXEC sp_rename 'fifa_data.height', 'height_in_cm', 'COLUMN'

EXEC sp_rename 'fifa_data.weight', 'weight_in_lbs', 'COLUMN'

EXEC sp_rename 'fifa_data.LongName', 'Player_full_name', 'COLUMN'

EXEC sp_rename 'fifa_data.POT', 'POT(potential_rating)', 'COLUMN'

EXEC sp_rename 'fifa_data.BOV', 'BOV(best_overall_rating)', 'COLUMN'

EXEC sp_rename 'fifa_data.PAC', 'PAC(pace)', 'COLUMN'

EXEC sp_rename 'fifa_data.SHO', 'SHO(shooting)', 'COLUMN'

EXEC sp_rename 'fifa_data.PAS', 'PAS(passing)', 'COLUMN'
 
EXEC sp_rename 'fifa_data.DRI', 'DRI(dribbling)', 'COLUMN'
 
EXEC sp_rename 'fifa_data.DEF', 'DEF(defending)', 'COLUMN' 
 
EXEC sp_rename 'fifa_data.PHY', 'PHY(physicality)', 'COLUMN'
 
```

---

### 6. **Conclusion**

This project helped me learn other SQL syntax I was not exposed to earlier. Being my first project with SQL. I will appreciate your honest feedback. You can check out the new and clean dataset below

![](https://github.com/Berry-of-Tech/FIFA21-data-cleaning-project/blob/main/Clean%20fifa%20data%201.png)

![](https://github.com/Berry-of-Tech/FIFA21-data-cleaning-project/blob/main/Clean%20fifa%20data%202.png)

![](https://github.com/Berry-of-Tech/FIFA21-data-cleaning-project/blob/main/Clean%20fifa%20data%203.png)

![](https://github.com/Berry-of-Tech/FIFA21-data-cleaning-project/blob/main/Clean%20fifa%20data%204.png)

---

### 7. **Challenges encountered during the cleaning process**

I. The first time I thought I was done with the cleaning, I discovered some data are missing in the club and hits column which make me curious. I later realized I imported the dataset with the Latin code name instead of the UTF-8 code name, though I don't know anything about codenames until I have these issues
II. I had to drop my tables twice and import the complete data because of errors and importing using the wrong code name.
You can connect with me on:

- [My linkedIn Profile](https://www.linkedin.com/in/alajede-mustapha-6071211a9/ "My LinkedIn Profile")

- [My Twitter Profile](https://twitter.com/Berry_Analyst "My Twitter Profile")

Thanks for Reading!!!

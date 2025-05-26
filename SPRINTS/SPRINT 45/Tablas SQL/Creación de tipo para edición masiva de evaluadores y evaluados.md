```sql
CREATE TYPE AssessedFormTypeList AS TABLE
(
    nId_Assessed INT,
    nId_FormType INT
);
CREATE TYPE QuestionsAnswers AS TABLE
(
    nId_Question INT,
    sTextAnswer VARCHAR(MAX),
	nNumberAnswer INT,
	sOptions VARCHAR(MAX)
);
```
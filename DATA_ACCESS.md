# Data Access

This repository does **not** redistribute KKBox competition data.

## Source
Kaggle competition:

**WSDM - KKBox's Churn Prediction Challenge**

Competition slug used by the notebook:

```text
kkbox-churn-prediction-challenge
```

## Required files

The notebook downloads only:

```text
members_v3.csv.7z
train_v2.csv.7z
transactions.csv.7z
user_logs.csv.7z
```

## Access steps

1. Create/sign in to a Kaggle account.
2. Open the KKBox competition page.
3. Accept the competition rules.
4. Generate a Kaggle API access token.
5. Run the notebook in Google Colab.
6. Paste the token into the secure `getpass` prompt.

The token is written only to the temporary Colab runtime credential location and is not stored in this repository.

## Important
Do not commit:
- `.csv`
- `.7z`
- Kaggle tokens
- `kaggle.json`
- `access_token`
- row-level exports containing `msno`

The repository's MIT license applies to the original code and documentation only, not to KKBox/Kaggle data.

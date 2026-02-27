🏠 Bengaluru House Price Prediction (End-to-End ML Deployment)
📌 Project Summary

Built a production-style machine learning system to predict residential property prices in Bengaluru using structured real estate data.

This project goes beyond model training and includes:

Advanced data cleaning and feature engineering

Systematic outlier detection

Cross-validated model selection and hyperparameter tuning

REST API backend using Flask

Production deployment on AWS EC2 using Gunicorn

The objective was to build a deployable ML service rather than a notebook-only model.

🧹 Data Cleaning & Feature Engineering

All preprocessing and modeling were performed in Jupyter Notebook using Pandas and scikit-learn.

1. Dataset Loading & Preservation

Imported CSV dataset into Jupyter

Created a working copy to preserve raw data integrity

Performed all transformations on a separate cleaned dataframe

This prevents accidental mutation of source data.

2. Feature Pruning

Removed columns that were:

Irrelevant for price prediction

Highly sparse (mostly null values)

Redundant or noisy

This reduced dimensionality and improved model clarity.

3. Missing Value Handling

Identified null values using isnull().sum()

Dropped rows where necessary

Replaced numerical nulls using median imputation

Median was chosen over mean to reduce distortion from extreme values.

4. Standardizing Bedroom Information (BHK Creation)

The size column contained inconsistent formats:

"4 Bedroom"

"4 BHK"

Solution:

Tokenized strings

Extracted numeric component

Converted to integer

Created standardized bhk column

df_clean['bhk'] = df_clean['size'].apply(lambda x: int(x.split(' ')[0]))

This normalized structural property information across the dataset.

5. Cleaning total_sqft

The dataset contained:

Ranges (e.g., "2100-2850")

Non-numeric values

Created a transformation function to:

Convert valid floats

Convert ranges to their average

Remove invalid entries

def convert_sqft_to_num(x):
    tokens = x.split('-')
    if len(tokens) == 2:
        return (float(tokens[0]) + float(tokens[1])) / 2
    try:
        return float(x)
    except:
        return None

This ensured consistent numerical formatting.

6. Derived Feature: Price Per Square Foot

Created:

price_per_sqft = (price * 100000) / total_sqft

This feature was critical for detecting pricing anomalies and performing outlier analysis.

7. Location Dimensionality Reduction

The dataset had a high number of unique location values.

Steps:

Counted entries per location

Grouped locations with fewer than 10 data points into "other"

This reduced high-cardinality categorical noise and improved model stability.

8. Multi-Level Outlier Detection

Applied both logical and statistical filters.

A. Unrealistic Square Footage per BHK

Removed properties where:

total_sqft / bhk < 300

These were physically unrealistic configurations.

B. Price Per Sqft Statistical Filtering

For each location:

Computed mean price_per_sqft

Computed standard deviation

Removed values outside 1 standard deviation

This localized anomaly detection to prevent cross-location distortion.

C. Illogical BHK Pricing

Removed cases where:

Lower BHK properties were priced significantly higher

Compared to higher BHK properties in the same location

This enforced logical pricing consistency.

D. Bathroom Constraint

Removed properties where:

bath > bhk + 2

These were deemed unrealistic entries.

9. Final Feature Selection

Dropped intermediate columns used only for cleaning:

price_per_sqft

size

Final model features:

location

total_sqft

bath

bhk

🤖 Model Development & Selection
1. One-Hot Encoding

Converted categorical location column using:

pd.get_dummies()

To prevent multicollinearity (Dummy Variable Trap):

Dropped one encoded column

2. Train-Test Split

80% Training

20% Testing

Used sklearn's train_test_split to maintain separation between training and evaluation data.

3. Baseline Model: Linear Regression

Trained Linear Regression model and evaluated using R² score.

This established baseline performance.

4. Cross-Validation

Applied 5-fold cross-validation to:

Reduce overfitting

Improve generalization confidence

Validate model robustness

5. Model Comparison & Hyperparameter Tuning

Used GridSearchCV to evaluate:

Linear Regression

Lasso Regression

Decision Tree Regressor

GridSearchCV:

Explored hyperparameter combinations

Compared models systematically

Selected best model based on cross-validation performance

This automated model selection and reduced manual bias.

6. Production Prediction Function

Implemented a predict_price() function that:

Accepts:

location

total_sqft

bath

bhk

Converts inputs into properly structured feature vectors

Returns predicted house price

This function serves as the backend inference engine.

🌐 Backend Architecture

Structured project into:

/model
/server
/client

server Module
server.py

Defines REST API endpoints

Handles POST and GET requests

Calls prediction function

Returns JSON responses

Built using Flask.

util.py

Contains:

Model loading function

Location loading function

Prediction logic

Model is loaded once during server startup to avoid repeated disk I/O.

🚀 AWS EC2 Deployment
1. EC2 Instance Setup

Ubuntu Server (t2.micro)

Security group configured to allow:

Port 22 (SSH)

Port 5000 (Application)

Access restricted to personal IP for security

2. Server Configuration

SSH into instance:

ssh -i key.pem ubuntu@<ec2-public-ip>

Installed dependencies:

Python3

pip

Flask

NumPy

Pandas

scikit-learn

Gunicorn

3. Application Deployment

Uploaded project via:

Git clone from GitHub
OR

SCP file transfer

4. Production Server with Gunicorn

Instead of Flask development server:

gunicorn server:app

Gunicorn provides:

WSGI production serving

Concurrent request handling

Improved reliability

5. Production Hardening (Extended)

For a more robust deployment:

Configure Nginx as reverse proxy

Bind Gunicorn to localhost

Forward traffic through Nginx

Enable HTTPS via Let's Encrypt

Configure systemd service for auto-restart

Set up UFW firewall rules

This transforms the project from demo-level to production-grade.

6. Application Access

Accessible via:

http://<EC2-Public-IP>:5000

Or via domain if Nginx configured.

📈 Technical Highlights

Extensive real-world data cleaning

Multi-level outlier detection

Cross-validated model comparison

Hyperparameter optimization with GridSearchCV

RESTful API integration

Cloud deployment on AWS EC2

Production server setup with Gunicorn

🧠 Key Takeaways

Data cleaning and feature engineering significantly impact model performance

Cross-validation improves reliability of evaluation

Hyperparameter tuning reduces model bias

Deployment requires infrastructure knowledge beyond ML

Building full-stack ML systems requires both data science and DevOps skills
<h1 align="center" style="font-weight: bold;">Bengaluru House Price Prediction API 🏠</h1>

<p align="center">
  <a href="#overview">Overview</a> •
  <a href="#tech">Technologies</a> •
  <a href="#structure">Project Structure</a> •
  <a href="#pipeline">Data Pipeline</a> •
  <a href="#training">Model Training</a> •
  <a href="#backend">Backend</a> •
  <a href="#routes">API Endpoints</a> •
  <a href="#deployment">Deployment</a> •
  <a href="#takeaways">Skills / Takeaways</a>
</p>

<p align="center">
  <b>
    A full ML pipeline + Flask API that predicts Bengaluru home prices using location, total_sqft, bhk, and bath.
    Includes cleaning, outlier removal, model selection with GridSearchCV, and AWS EC2 deployment.
  </b>
</p>

<hr>

<h2 id="overview">📌 Overview</h2>

<ul>
  <li><b>Goal:</b> Predict Bengaluru house prices based on location + property features.</li>
  <li><b>Inputs:</b> location, total_sqft, bhk, bath</li>
  <li><b>Output:</b> estimated price</li>
  <li><b>No frontend:</b> API-only (tested with Postman).</li>
</ul>

<hr>

<h2 id="tech">💻 Technologies</h2>

<ul>
  <li>Python</li>
  <li>Jupyter Notebook</li>
  <li>Pandas / NumPy</li>
  <li>scikit-learn (Linear Regression, cross-validation, GridSearchCV)</li>
  <li>Flask (REST API)</li>
  <li>Gunicorn (production server)</li>
  <li>AWS EC2 (Ubuntu)</li>
  <li>Postman (API testing)</li>
</ul>

<hr>

<h2 id="structure">📂 Project Structure</h2>

<pre>
.
├── model/
│   ├── bangalore_home_prices_model.pickle
│   └── columns.json
│
├── server/
│   ├── server.py
│   └── util.py
│
└── notebook/
    └── data_cleaning_and_training.ipynb
</pre>

<hr>

<h2 id="pipeline">🧹 Data Cleaning & Feature Engineering (Notebook)</h2>

<p>
All preprocessing was done in Jupyter Notebook. Cleaning steps were applied using new DataFrames when appropriate to
avoid accidentally modifying the original raw data.
</p>

<h3>1) Load Dataset</h3>
<ul>
  <li>Import CSV into Jupyter Notebook</li>
  <li>Create a Pandas DataFrame</li>
  <li>Drop unnecessary columns to simplify the dataset</li>
  <li>Create a new dataset/DataFrame for cleaning so the initial data remains unchanged</li>
</ul>

<h3>2) Handle Missing Values</h3>
<ul>
  <li>Identify null values</li>
  <li>Drop rows with null values or replace nulls using the <b>median</b> of the column</li>
</ul>

<h3>3) Unify Bedroom Format (BHK)</h3>
<ul>
  <li>Unify entries like “4 Bedrooms” and “4 BHK” into a single numeric representation</li>
  <li>Tokenize the <code>size</code> string</li>
  <li>Use a lambda function to extract the first token, convert to integer, and store it in a new <code>bhk</code> column</li>
</ul>

<h3>4) Unify and Convert <code>total_sqft</code></h3>
<ul>
  <li>Some <code>total_sqft</code> values are not floats (strings, ranges, etc.)</li>
  <li>Write a function to identify values that cannot be converted to float</li>
  <li>For ranges (example: <code>"1200 - 1500"</code>), write a function that returns the <b>average</b> of the two values</li>
  <li>Drop values that cannot be reliably converted</li>
</ul>

<h3>5) Create <code>price_per_sqft</code> (Outlier Detection)</h3>
<ul>
  <li>Create a new DataFrame and add <code>price_per_sqft</code> to support outlier cleaning</li>
</ul>

<h3>6) Location Cleanup</h3>
<ul>
  <li>Check number of unique locations and distribution</li>
  <li>Group all locations with fewer than <b>10</b> data points into a new category called <code>other</code></li>
</ul>

<hr>

<h2 id="training">📈 Outlier Removal + Model Training</h2>

<h3>Outlier Removal</h3>
<ul>
  <li>Create new DataFrames during outlier removal to keep steps clean and reproducible</li>
  <li>Remove unrealistic records where <code>total_sqft / bhk &lt; 300</code></li>
  <li>For each location, compute mean and standard deviation of <code>price_per_sqft</code> and filter out points beyond that deviation</li>
  <li>Remove cases where a home with fewer BHK has a higher price than a home with more BHK in the same area (inconsistent pricing patterns)</li>
  <li>Remove bathroom outliers: drop properties with <b>2 or more bathrooms than BHK</b></li>
  <li>Drop columns not needed for training (example: <code>price_per_sqft</code>, and <code>size</code> since <code>bhk</code> is used)</li>
</ul>

<h3>Feature Encoding</h3>
<ul>
  <li>One-hot encode <code>location</code></li>
  <li>Drop one dummy column to avoid <b>multicollinearity</b></li>
</ul>

<h3>Train/Test Split</h3>
<ul>
  <li>Split dataset into X and y:
    <ul>
      <li><b>y (dependent):</b> price</li>
      <li><b>X (independent):</b> all other features</li>
    </ul>
  </li>
  <li>80% train, 20% test</li>
</ul>

<h3>Baseline Model</h3>
<ul>
  <li>Train a Linear Regression model using scikit-learn</li>
  <li>Evaluate using model score</li>
</ul>

<h3>Cross Validation</h3>
<ul>
  <li>Perform 5-fold cross validation and evaluate the model across folds</li>
</ul>

<h3>GridSearchCV (Model Selection)</h3>
<ul>
  <li>Use GridSearchCV to compare multiple models and parameter combinations</li>
  <li>Select the best model and best hyperparameters based on score</li>
</ul>

<h3>Prediction Function</h3>
<ul>
  <li>Write a final prediction function that accepts location, sqft, bhk, and bath and returns estimated price</li>
  <li>This prediction logic is reused inside the backend API</li>
</ul>

<hr>

<h2 id="backend">🧩 Backend (Flask API)</h2>

<p>
The backend exposes the trained model through REST endpoints. The backend is organized into <code>server.py</code> and <code>util.py</code>.
</p>

<h3>server/</h3>
<ul>
  <li><code>server.py</code>: defines API routes and handles requests/responses</li>
  <li><code>util.py</code>: contains model loading + prediction helper functions</li>
</ul>

<h3>util.py Functions</h3>
<ul>
  <li><b>Prediction function</b>: same logic used during training for consistent results</li>
  <li><b>Load artifacts</b>: loads the trained model + required metadata</li>
  <li><b>Read dataset values</b>: returns supported locations / columns needed for prediction</li>
</ul>

<p>All endpoints were tested using Postman.</p>

<hr>

<h2 id="routes">📍 API Endpoints</h2>

<table>
  <thead>
    <tr>
      <th>Route</th>
      <th>Description</th>
      <th>Request Body</th>
      <th>Response</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><kbd>GET /get_location_names</kbd></td>
      <td>Returns supported locations</td>
      <td>None</td>
      <td>JSON list of locations</td>
    </tr>
    <tr>
      <td><kbd>POST /predict_home_price</kbd></td>
      <td>Returns predicted house price</td>
      <td>location, total_sqft, bhk, bath</td>
      <td>estimated_price</td>
    </tr>
  </tbody>
</table>

<h3>GET /get_location_names</h3>

<p><b>RESPONSE</b></p>
<pre><code class="language-json">
{
  "locations": ["Indira Nagar", "Whitefield", "Electronic City", "other"]
}
</code></pre>

<h3>POST /predict_home_price</h3>

<p><b>REQUEST</b></p>
<pre><code class="language-json">
{
  "location": "Indira Nagar",
  "total_sqft": 1200,
  "bhk": 2,
  "bath": 2
}
</code></pre>

<p><b>RESPONSE</b></p>
<pre><code class="language-json">
{
  "estimated_price": 85.6
}
</code></pre>

<hr>

<h2 id="deployment">☁️ Deployment (AWS EC2)</h2>

<ul>
  <li>Create and launch an Ubuntu EC2 instance with security permissions allowing port <b>5000</b> from your personal IP address</li>
  <li>Connect to the EC2 instance via SSH using your terminal</li>
  <li>Upload the project folder to the EC2 instance</li>
  <li>Install Python dependencies and <b>gunicorn</b> on the EC2 instance</li>
  <li>Run the Flask server (commonly via gunicorn)</li>
  <li>Use the EC2 public URL/IP to access the API</li>
</ul>

<hr>

<h2 id="takeaways">🎯 Skills Learned & Key Takeaways</h2>

<ul>
  <li><b>Data Cleaning:</b> Cleaned real-world messy data (missing values, mixed formats, range parsing).</li>
  <li><b>Feature Engineering:</b> Standardized text-based fields (like <code>size</code>) into numeric model-ready features (<code>bhk</code>).</li>
  <li><b>Outlier Detection:</b> Applied both logical rules (sqft/BHK, bathroom constraints) and statistical filtering (mean + std by location).</li>
  <li><b>Model Validation:</b> Used train/test split and 5-fold cross validation to measure generalization.</li>
  <li><b>Hyperparameter Tuning:</b> Used GridSearchCV to compare models and select best hyperparameters.</li>
  <li><b>Productionizing ML:</b> Saved trained artifacts and reused them consistently for inference.</li>
  <li><b>API Development:</b> Built and tested REST endpoints with Flask and Postman.</li>
  <li><b>Cloud Deployment:</b> Deployed a production-style backend using Gunicorn on AWS EC2.</li>
  <li><b>End-to-End Workflow:</b> Completed the full path from raw dataset → trained model → live prediction API.</li>
</ul>

<hr>

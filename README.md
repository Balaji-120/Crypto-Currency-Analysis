# Crypto-Currency-Analysis
# Import necessary libraries
import requests
import pandas as pd
import matplotlib.pyplot as plt
import seaborn as sns

# Configure matplotlib for Jupyter Notebook
%matplotlib inline

# Function to fetch historical cryptocurrency data
def fetch_crypto_data(crypto_id="bitcoin", vs_currency="usd", days="30"):
    """
    Fetch historical cryptocurrency price and volume data from CoinGecko API.
    :param crypto_id: The ID of the cryptocurrency (e.g., "bitcoin").
    :param vs_currency: The currency for comparison (e.g., "usd").
    :param days: Number of past days to fetch data for (e.g., "30").
    :return: DataFrame containing timestamp, price, and volume.
    """
    url = f"https://api.coingecko.com/api/v3/coins/{crypto_id}/market_chart"
    params = {"vs_currency": vs_currency, "days": days}
    response = requests.get(url, params=params)
    if response.status_code == 200:
        data = response.json()
        prices = data["prices"]
        volumes = data["total_volumes"]
        df = pd.DataFrame(prices, columns=["timestamp", "price"])
        df["volume"] = pd.DataFrame(volumes)[1]
        df["timestamp"] = pd.to_datetime(df["timestamp"], unit="ms")
        return df
    else:
        raise Exception("Failed to fetch data. Please check the API or parameters.")

# Fetch data
crypto_data = fetch_crypto_data(crypto_id="bitcoin", vs_currency="usd", days="90")

# Display first few rows
crypto_data.head()
# Check for missing values
print("Missing values:\n", crypto_data.isnull().sum())

# Basic statistics
print("Basic Statistics:\n", crypto_data.describe())

# Set timestamp as the index for better visualization
crypto_data.set_index("timestamp", inplace=True)
# Plot price trends
plt.figure(figsize=(14, 7))
plt.plot(crypto_data.index, crypto_data["price"], label="Price (USD)")
plt.title("Bitcoin Price Trend (Last 90 Days)")
plt.xlabel("Date")
plt.ylabel("Price (USD)")
plt.legend()
plt.grid()
plt.show()
# Plot trading volume
plt.figure(figsize=(14, 7))
plt.bar(crypto_data.index, crypto_data["volume"], color="blue", alpha=0.5, label="Volume")
plt.title("Bitcoin Trading Volume (Last 90 Days)")
plt.xlabel("Date")
plt.ylabel("Volume")
plt.legend()
plt.grid()
plt.show()
# Add moving averages to the DataFrame
crypto_data["7-day MA"] = crypto_data["price"].rolling(window=7).mean()
crypto_data["30-day MA"] = crypto_data["price"].rolling(window=30).mean()

# Plot moving averages with price
plt.figure(figsize=(14, 7))
plt.plot(crypto_data.index, crypto_data["price"], label="Price (USD)", alpha=0.6)
plt.plot(crypto_data.index, crypto_data["7-day MA"], label="7-day Moving Average", color="orange")
plt.plot(crypto_data.index, crypto_data["30-day MA"], label="30-day Moving Average", color="red")
plt.title("Bitcoin Price with Moving Averages")
plt.xlabel("Date")
plt.ylabel("Price (USD)")
plt.legend()
plt.grid()
plt.show()
# Calculate daily returns
crypto_data["daily_return"] = crypto_data["price"].pct_change()
# Plot histogram of daily returns
plt.figure(figsize=(14, 7))
sns.histplot(crypto_data["daily_return"].dropna(), bins=50, kde=True, color="purple")
plt.title("Histogram of Daily Returns")
plt.xlabel("Daily Return")
plt.ylabel("Frequency")
plt.grid()
plt.show()

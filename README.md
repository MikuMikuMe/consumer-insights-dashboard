# consumer-insights-dashboard

To create a consumer insights dashboard, we will develop a scalable and interactive web application using Python. We'll utilize Flask as our web framework, Plotly for interactive visualizations, and Pandas for data manipulation. This example assumes that we're consuming data from a simulated real-time data source. 

Here's a complete step-by-step Python program to achieve this:

```python
from flask import Flask, render_template, jsonify, request
import pandas as pd
import plotly
import plotly.express as px
from datetime import datetime
import json
import random
import threading
import time

app = Flask(__name__)

# Create a lock for thread safety
data_lock = threading.Lock()

# Simulate initial data
data = pd.DataFrame({
    'timestamp': [datetime.now()],
    'customer_id': [1],
    'action': ['start']
})

# Simulated real-time data update
def update_data():
    while True:
        time.sleep(5)  # Simulate a 5-second interval between data updates
        with data_lock:
            global data
            new_data = {
                'timestamp': datetime.now(),
                'customer_id': random.randint(1, 100),
                'action': random.choice(['view', 'click', 'purchase'])
            }
            data = data.append(new_data, ignore_index=True)
            print(f"Data updated: {new_data}")

# Set up and start data update thread
data_update_thread = threading.Thread(target=update_data, daemon=True)
data_update_thread.start()

# Route for the main dashboard
@app.route('/')
def index():
    return render_template('index.html')

# Route to fetch the real-time data for visualization
@app.route('/get_data')
def get_data():
    with data_lock:
        try:
            # Return the data as JSON
            return jsonify(data.to_dict(orient='records'))
        except Exception as e:
            print(f"Error retrieving data: {e}")
            return jsonify({'error': str(e)}), 500

# Route for generating Plotly visualizations
@app.route('/plot')
def plot():
    with data_lock:
        try:
            # Data processing
            latest_data = data.copy()
            agg_data = latest_data.groupby('action').size().reset_index(name='count')
            
            # Generate Plotly figure
            fig = px.bar(agg_data, x='action', y='count', title='Customer Actions')
            
            # Return figure as JSON
            graph_json = json.dumps(fig, cls=plotly.utils.PlotlyJSONEncoder)
            return graph_json
        except Exception as e:
            print(f"Error generating plot: {e}")
            return jsonify({'error': str(e)}), 500

# Main entry point of the application
if __name__ == '__main__':
    try:
        app.run(debug=True)
    except Exception as e:
        print(f"Error starting Flask app: {e}")
```

Template `index.html` (ensure this file is in a folder named `templates`):
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Consumer Insights Dashboard</title>
    <script src="https://cdn.plot.ly/plotly-latest.min.js"></script>
</head>
<body>
    <h1>Consumer Insights Dashboard</h1>
    <div id="plot"></div>

    <script>
      // Fetch plot data from server and render using Plotly.js
      function fetchData() {
        fetch('/plot')
          .then(response => response.json())
          .then(data => {
            Plotly.newPlot('plot', data.data, data.layout);
          })
          .catch(error => console.log('Error loading plot: ' + error));
      }
      
      // Periodically refresh the plot
      setInterval(fetchData, 5000);
      fetchData();
    </script>
</body>
</html>
```

### Explanation:

1. **Flask**: We use Flask to create a web server with endpoints to serve the dashboard and data.

2. **Data Simulation**: A separate thread simulates the real-time data updates every 5 seconds.

3. **Routes**:
   - `/`: Renders the main web page containing the dashboard.
   - `/get_data`: Provides a JSON API endpoint to retrieve data.
   - `/plot`: Produces a Plotly bar chart JSON object from the aggregated data.

4. **Multithreading**: A thread simulates incoming data in real-time, ensuring a continuous data flow.

5. **Error Handling**: Try-except blocks handle errors gracefully, logging them, and sending meaningful error messages to the client.

6. **Template**: A simple HTML template uses Plotly.js to render the graph, which auto-refreshes every 5 seconds.

Remember to install Flask and Plotly if you haven't yet:
```bash
pip install Flask pandas plotly
```

This program provides a basic starting point for a real-time consumer insights dashboard, suitable for further expansion and real data integration.
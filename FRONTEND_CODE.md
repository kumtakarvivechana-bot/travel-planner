# Travel Planner Frontend Code

Complete frontend implementation for the Intelligent Travel & Budget Planner web application.

## File Structure
```
travel_planner/
├── app.py                 # Flask backend server
├── templates/
│   └── index.html        # Main HTML template
├── static/
│   ├── style.css         # Styling
│   └── script.js         # JavaScript functionality
└── src/
    ├── constraints.py    # Constraint validation
    ├── agent.py          # Travel planner agent
    ├── itinerary.py      # Itinerary generation
    └── scenarios.py      # Scenario generation
```

---

## 1. HTML Template (templates/index.html)

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Intelligent Travel & Budget Planner</title>
    <link rel="stylesheet" href="{{ url_for('static', filename='style.css') }}">
</head>
<body>
    <div class="container">
        <!-- Header -->
        <header class="header">
            <div class="header-content">
                <h1>✈️ Travel & Budget Planner</h1>
                <p>Plan your perfect trip with smart budget management</p>
            </div>
        </header>

        <!-- Main Content -->
        <main class="main-content">
            <!-- Step 1: Constraints -->
            <section class="section" id="constraints-section">
                <h2>Step 1: Define Your Trip Constraints</h2>
                
                <button class="btn btn-demo" onclick="runDemoWorkflow()" style="margin-bottom: 20px;">
                    🎬 Run Full Demo (Auto-fill & Create Plan)
                </button>
                
                <div class="form-grid">
                    <div class="form-group">
                        <label>Total Budget ($)</label>
                        <input type="number" id="budget" value="1500" min="100" step="100">
                    </div>
                    <div class="form-group">
                        <label>Duration (days)</label>
                        <input type="number" id="duration" value="5" min="1" max="30">
                    </div>
                    <div class="form-group">
                        <label>Accommodation Quality (1-5)</label>
                        <input type="number" id="accommodation_quality" value="3" min="1" max="5">
                    </div>
                    <div class="form-group">
                        <label>Transportation</label>
                        <select id="transportation">
                            <option value="public">Public Transport</option>
                            <option value="private">Private/Car</option>
                            <option value="mixed">Mixed</option>
                        </select>
                    </div>
                </div>

                <div class="form-group">
                    <label>Must-Visit Locations (comma-separated)</label>
                    <input type="text" id="must_visit" placeholder="e.g., Eiffel Tower, Louvre Museum">
                </div>

                <div class="form-group">
                    <label>Activities to Avoid (comma-separated)</label>
                    <input type="text" id="avoid_activities" placeholder="e.g., Skydiving, Extreme Sports">
                </div>

                <button class="btn btn-primary" onclick="createAgent()">Create Trip Plan</button>
                <div id="constraint-message" class="message"></div>
            </section>

            <!-- Step 2: Add Activities -->
            <section class="section" id="activities-section" style="display:none;">
                <h2>Step 2: Add Activities</h2>
                
                <div class="form-grid">
                    <div class="form-group">
                        <label>Activity Name</label>
                        <input type="text" id="activity_name" placeholder="e.g., Eiffel Tower Visit">
                    </div>
                    <div class="form-group">
                        <label>Activity Type</label>
                        <select id="activity_type" onchange="updateActivityType()">
                            <option value="sightseeing">Sightseeing</option>
                            <option value="accommodation">Accommodation</option>
                            <option value="dining">Dining</option>
                            <option value="transportation">Transportation</option>
                            <option value="entertainment">Entertainment</option>
                            <option value="activity">Activity/Adventure</option>
                        </select>
                    </div>
                    <div class="form-group">
                        <label>Cost ($)</label>
                        <input type="number" id="activity_cost" value="0" min="0" step="0.01">
                    </div>
                    <div class="form-group">
                        <label>Duration (hours)</label>
                        <input type="number" id="activity_duration" value="2" min="1" step="0.5">
                    </div>
                    <div class="form-group">
                        <label>Time Slot</label>
                        <select id="activity_time">
                            <option value="morning">Morning</option>
                            <option value="afternoon">Afternoon</option>
                            <option value="evening">Evening</option>
                            <option value="all-day">All Day</option>
                        </select>
                    </div>
                    <div class="form-group">
                        <label>Priority (1-5)</label>
                        <input type="number" id="activity_priority" value="3" min="1" max="5">
                    </div>
                </div>

                <div class="form-group">
                    <label>Description</label>
                    <textarea id="activity_description" rows="3" placeholder="Brief description of the activity"></textarea>
                </div>

                <button class="btn btn-success" onclick="addActivity()">Add Activity</button>
                <div id="activity-message" class="message"></div>

                <!-- Activities List -->
                <div class="activities-list" id="activities-list">
                    <h3>Added Activities (<span id="activity-count">0</span>)</h3>
                    <div id="activities-container" class="activities-container"></div>
                </div>
            </section>

            <!-- Step 3: Create Plan -->
            <section class="section" id="plan-section" style="display:none;">
                <h2>Step 3: Generate Your Travel Plan</h2>
                
                <div class="form-group">
                    <label>Start Date</label>
                    <input type="date" id="start_date">
                </div>

                <button class="btn btn-primary" onclick="createPlan()">Create Plan</button>
                <div id="plan-message" class="message"></div>

                <!-- Itinerary Display -->
                <div id="itinerary-display" style="display:none;">
                    <div class="tabs">
                        <button class="tab-btn active" onclick="switchTab('markdown')">Itinerary</button>
                        <button class="tab-btn" onclick="switchTab('summary')">Summary</button>
                        <button class="tab-btn" onclick="switchTab('json')">JSON</button>
                    </div>

                    <div id="markdown-tab" class="tab-content active">
                        <div id="itinerary-markdown" class="markdown-content"></div>
                    </div>

                    <div id="summary-tab" class="tab-content">
                        <div id="itinerary-summary" class="summary-content"></div>
                    </div>

                    <div id="json-tab" class="tab-content">
                        <pre id="itinerary-json"></pre>
                    </div>
                </div>
            </section>

            <!-- Step 4: Scenarios -->
            <section class="section" id="scenarios-section" style="display:none;">
                <h2>Step 4: Explore Alternative Scenarios</h2>
                
                <div class="scenario-buttons">
                    <button class="btn btn-info" onclick="generateScenarios('budget')">Budget-Based Scenarios</button>
                    <button class="btn btn-info" onclick="generateScenarios('priority')">Priority-Based Scenarios</button>
                    <button class="btn btn-info" onclick="generateScenarios('cost_reduction')">Cost-Reduction Strategies</button>
                </div>

                <div id="scenarios-display" style="display:none;">
                    <div id="scenarios-comparison" class="scenarios-content"></div>
                </div>
            </section>
        </main>

        <!-- Footer -->
        <footer class="footer">
            <p>&copy; 2025 Intelligent Travel & Budget Planner | Plan Smart, Travel Happy</p>
        </footer>
    </div>

    <script src="{{ url_for('static', filename='script.js') }}"></script>
</body>
</html>
```

---

## 2. CSS Styling (static/style.css)

```css
/* Travel Planner - Main Styles */

* {
    margin: 0;
    padding: 0;
    box-sizing: border-box;
}

:root {
    --primary-color: #4CAF50;
    --secondary-color: #2196F3;
    --danger-color: #f44336;
    --warning-color: #ff9800;
    --light-gray: #f5f5f5;
    --dark-gray: #333;
    --border-radius: 8px;
    --box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
}

body {
    font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
    background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
    color: var(--dark-gray);
    line-height: 1.6;
    min-height: 100vh;
}

.container {
    max-width: 1200px;
    margin: 0 auto;
    padding: 20px;
}

/* Header */
.header {
    background: white;
    border-radius: var(--border-radius);
    padding: 40px;
    margin-bottom: 30px;
    box-shadow: var(--box-shadow);
    text-align: center;
}

.header h1 {
    color: var(--primary-color);
    font-size: 2.5em;
    margin-bottom: 10px;
}

.header p {
    color: #666;
    font-size: 1.1em;
}

/* Main Content */
.main-content {
    display: flex;
    flex-direction: column;
    gap: 20px;
    margin-bottom: 40px;
}

/* Sections */
.section {
    background: white;
    border-radius: var(--border-radius);
    padding: 30px;
    box-shadow: var(--box-shadow);
    animation: slideIn 0.3s ease-in;
}

@keyframes slideIn {
    from {
        opacity: 0;
        transform: translateY(10px);
    }
    to {
        opacity: 1;
        transform: translateY(0);
    }
}

.section h2 {
    color: var(--primary-color);
    margin-bottom: 20px;
    padding-bottom: 10px;
    border-bottom: 2px solid var(--light-gray);
}

/* Forms */
.form-grid {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
    gap: 20px;
    margin-bottom: 20px;
}

.form-group {
    display: flex;
    flex-direction: column;
    gap: 8px;
}

.form-group label {
    font-weight: 600;
    color: var(--dark-gray);
}

.form-group input,
.form-group select,
.form-group textarea {
    padding: 10px;
    border: 1px solid #ddd;
    border-radius: 4px;
    font-size: 1em;
    font-family: inherit;
    transition: border-color 0.3s;
}

.form-group input:focus,
.form-group select:focus,
.form-group textarea:focus {
    outline: none;
    border-color: var(--primary-color);
    box-shadow: 0 0 5px rgba(76, 175, 80, 0.3);
}

/* Buttons */
.btn {
    padding: 12px 24px;
    border: none;
    border-radius: 4px;
    font-size: 1em;
    font-weight: 600;
    cursor: pointer;
    transition: all 0.3s;
    text-transform: uppercase;
    letter-spacing: 0.5px;
}

.btn-primary {
    background: var(--primary-color);
    color: white;
}

.btn-primary:hover {
    background: #45a049;
    box-shadow: 0 4px 12px rgba(76, 175, 80, 0.4);
}

.btn-success {
    background: var(--primary-color);
    color: white;
}

.btn-success:hover {
    background: #45a049;
    box-shadow: 0 4px 12px rgba(76, 175, 80, 0.4);
}

.btn-info {
    background: var(--secondary-color);
    color: white;
}

.btn-info:hover {
    background: #0b7dda;
    box-shadow: 0 4px 12px rgba(33, 150, 243, 0.4);
}

.btn-demo {
    background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
    color: white;
}

.btn-demo:hover {
    background: linear-gradient(135deg, #5568d3 0%, #6a3d8f 100%);
    box-shadow: 0 4px 12px rgba(102, 126, 234, 0.4);
}

.btn:disabled {
    background: #ccc;
    cursor: not-allowed;
    opacity: 0.6;
}

/* Messages */
.message {
    margin-top: 15px;
    padding: 12px 16px;
    border-radius: 4px;
    display: none;
}

.message.success {
    display: block;
    background: #d4edda;
    color: #155724;
    border: 1px solid #c3e6cb;
}

.message.error {
    display: block;
    background: #f8d7da;
    color: #721c24;
    border: 1px solid #f5c6cb;
}

.message.info {
    display: block;
    background: #d1ecf1;
    color: #0c5460;
    border: 1px solid #bee5eb;
}

/* Activities List */
.activities-list {
    margin-top: 30px;
    border-top: 2px solid var(--light-gray);
    padding-top: 20px;
}

.activities-list h3 {
    color: var(--primary-color);
    margin-bottom: 15px;
}

.activities-container {
    display: grid;
    grid-template-columns: repeat(auto-fill, minmax(280px, 1fr));
    gap: 15px;
}

.activity-card {
    background: var(--light-gray);
    border-left: 4px solid var(--primary-color);
    padding: 15px;
    border-radius: 4px;
    position: relative;
}

.activity-card h4 {
    color: var(--primary-color);
    margin-bottom: 8px;
}

.activity-card p {
    font-size: 0.9em;
    color: #666;
    margin: 5px 0;
}

.activity-cost {
    font-weight: bold;
    color: var(--danger-color);
    font-size: 1.1em;
}

.activity-priority {
    display: inline-block;
    background: var(--warning-color);
    color: white;
    padding: 2px 8px;
    border-radius: 12px;
    font-size: 0.85em;
    margin-top: 8px;
}

/* Tabs */
.tabs {
    display: flex;
    gap: 10px;
    margin-bottom: 20px;
    border-bottom: 2px solid var(--light-gray);
}

.tab-btn {
    padding: 10px 20px;
    background: none;
    border: none;
    border-bottom: 3px solid transparent;
    cursor: pointer;
    font-weight: 600;
    color: #666;
    transition: all 0.3s;
}

.tab-btn.active {
    color: var(--primary-color);
    border-bottom-color: var(--primary-color);
}

.tab-btn:hover {
    color: var(--primary-color);
}

.tab-content {
    display: none;
}

.tab-content.active {
    display: block;
    animation: fadeIn 0.3s ease-in;
}

@keyframes fadeIn {
    from {
        opacity: 0;
    }
    to {
        opacity: 1;
    }
}

/* Content Displays */
.markdown-content {
    background: var(--light-gray);
    padding: 20px;
    border-radius: 4px;
    overflow-x: auto;
    font-family: 'Courier New', monospace;
    font-size: 0.95em;
    line-height: 1.8;
    white-space: pre-wrap;
    word-wrap: break-word;
}

.summary-content {
    display: grid;
    grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
    gap: 15px;
}

.summary-item {
    background: var(--light-gray);
    padding: 20px;
    border-radius: 4px;
    text-align: center;
    border-top: 4px solid var(--primary-color);
}

.summary-item-label {
    font-size: 0.9em;
    color: #666;
    margin-bottom: 8px;
}

.summary-item-value {
    font-size: 1.8em;
    font-weight: bold;
    color: var(--primary-color);
}

.summary-item-unit {
    font-size: 0.9em;
    color: #666;
}

/* JSON Display */
pre {
    background: var(--light-gray);
    padding: 20px;
    border-radius: 4px;
    overflow-x: auto;
    border-left: 4px solid var(--secondary-color);
    font-family: 'Courier New', monospace;
    font-size: 0.9em;
}

/* Scenarios */
.scenario-buttons {
    display: flex;
    gap: 10px;
    flex-wrap: wrap;
    margin-bottom: 20px;
}

.scenarios-content {
    background: var(--light-gray);
    padding: 20px;
    border-radius: 4px;
    font-family: 'Courier New', monospace;
    font-size: 0.9em;
    line-height: 1.8;
    white-space: pre-wrap;
    word-wrap: break-word;
    overflow-x: auto;
}

/* Footer */
.footer {
    background: white;
    border-radius: var(--border-radius);
    padding: 20px;
    text-align: center;
    color: #666;
    box-shadow: var(--box-shadow);
    margin-top: 40px;
}

/* Responsive */
@media (max-width: 768px) {
    .header h1 {
        font-size: 1.8em;
    }

    .form-grid {
        grid-template-columns: 1fr;
    }

    .activities-container {
        grid-template-columns: 1fr;
    }

    .tabs {
        flex-direction: column;
    }

    .scenario-buttons {
        flex-direction: column;
    }

    .btn {
        width: 100%;
    }
}

/* Loading Animation */
.spinner {
    border: 4px solid var(--light-gray);
    border-top: 4px solid var(--primary-color);
    border-radius: 50%;
    width: 40px;
    height: 40px;
    animation: spin 0.8s linear infinite;
    margin: 20px auto;
}

@keyframes spin {
    0% {
        transform: rotate(0deg);
    }
    100% {
        transform: rotate(360deg);
    }
}
```

---

## 3. JavaScript (static/script.js)

```javascript
// Travel Planner - JavaScript

let currentAgentId = null;
let activityCount = 0;

// Set today's date as default start date and load demo data
document.addEventListener('DOMContentLoaded', function() {
    const today = new Date().toISOString().split('T')[0];
    document.getElementById('start_date').value = today;
    
    // Load demo data on page load
    loadDemoData();
});

// Load Demo Data
function loadDemoData() {
    // Pre-fill form with demo values
    document.getElementById('budget').value = '1500';
    document.getElementById('duration').value = '4';
    document.getElementById('accommodation_quality').value = '3';
    document.getElementById('transportation').value = 'public';
    document.getElementById('must_visit').value = 'Times Square, Statue of Liberty';
    document.getElementById('avoid_activities').value = 'Skydiving';
    
    // Pre-fill activity form with first demo activity
    document.getElementById('activity_name').value = 'Hotel Accommodation (4 nights)';
    document.getElementById('activity_type').value = 'accommodation';
    document.getElementById('activity_cost').value = '500';
    document.getElementById('activity_duration').value = '1';
    document.getElementById('activity_time').value = 'all-day';
    document.getElementById('activity_priority').value = '5';
    document.getElementById('activity_description').value = 'Mid-range hotel in Manhattan';
}

// Create Agent with Constraints
async function createAgent() {
    const budget = parseFloat(document.getElementById('budget').value);
    const duration = parseInt(document.getElementById('duration').value);
    const accommodation_quality = parseInt(document.getElementById('accommodation_quality').value);
    const transportation = document.getElementById('transportation').value;
    
    const must_visit = document.getElementById('must_visit').value
        .split(',')
        .map(s => s.trim())
        .filter(s => s);
    
    const avoid_activities = document.getElementById('avoid_activities').value
        .split(',')
        .map(s => s.trim())
        .filter(s => s);
    
    const message = document.getElementById('constraint-message');
    
    try {
        message.className = 'message';
        message.textContent = 'Creating agent...';
        
        const response = await fetch('/api/create-agent', {
            method: 'POST',
            headers: {
                'Content-Type': 'application/json'
            },
            body: JSON.stringify({
                budget,
                duration,
                accommodation_quality,
                transportation,
                must_visit,
                avoid_activities
            })
        });
        
        const data = await response.json();
        
        if (data.success) {
            currentAgentId = data.agent_id;
            activityCount = 0;
            
            message.className = 'message success';
            message.textContent = `✓ Trip plan created! Budget: $${budget}, Duration: ${duration} days`;
            
            // Show next sections
            document.getElementById('activities-section').style.display = 'block';
            document.getElementById('plan-section').style.display = 'block';
            document.getElementById('scenarios-section').style.display = 'block';
            
            // Scroll to activities
            setTimeout(() => {
                document.getElementById('activities-section').scrollIntoView({ behavior: 'smooth' });
            }, 300);
        } else {
            message.className = 'message error';
            message.textContent = `✗ Error: ${data.error}`;
        }
    } catch (error) {
        message.className = 'message error';
        message.textContent = `✗ Error: ${error.message}`;
    }
}

// Add Activity
async function addActivity() {
    if (!currentAgentId) {
        showMessage('activity-message', 'error', 'Please create a trip plan first');
        return;
    }
    
    const name = document.getElementById('activity_name').value.trim();
    const type = document.getElementById('activity_type').value;
    const cost = parseFloat(document.getElementById('activity_cost').value);
    const duration = parseFloat(document.getElementById('activity_duration').value);
    const time_slot = document.getElementById('activity_time').value;
    const priority = parseInt(document.getElementById('activity_priority').value);
    const description = document.getElementById('activity_description').value.trim();
    
    if (!name) {
        showMessage('activity-message', 'error', 'Please enter activity name');
        return;
    }
    
    const message = document.getElementById('activity-message');
    
    try {
        message.className = 'message';
        message.textContent = 'Adding activity...';
        
        const response = await fetch('/api/add-activity', {
            method: 'POST',
            headers: {
                'Content-Type': 'application/json'
            },
            body: JSON.stringify({
                agent_id: currentAgentId,
                name,
                type,
                cost,
                duration,
                time_slot,
                priority,
                description
            })
        });
        
        const data = await response.json();
        
        if (data.success) {
            activityCount = data.activity_count;
            
            message.className = 'message success';
            message.textContent = `✓ Activity added! Total: ${activityCount} activities`;
            
            // Add to display list
            addActivityToList(data.activity);
            
            // Clear form
            document.getElementById('activity_name').value = '';
            document.getElementById('activity_cost').value = '0';
            document.getElementById('activity_duration').value = '2';
            document.getElementById('activity_description').value = '';
            
            // Update count
            document.getElementById('activity-count').textContent = activityCount;
        } else {
            message.className = 'message error';
            message.textContent = `✗ Error: ${data.error}`;
        }
    } catch (error) {
        message.className = 'message error';
        message.textContent = `✗ Error: ${error.message}`;
    }
}

// Add Activity to Display List
function addActivityToList(activity) {
    const container = document.getElementById('activities-container');
    
    const card = document.createElement('div');
    card.className = 'activity-card';
    card.innerHTML = `
        <h4>${activity.name}</h4>
        <p><strong>Type:</strong> ${activity.type}</p>
        <p><strong>Cost:</strong> <span class="activity-cost">$${activity.cost.toFixed(2)}</span></p>
        <p><strong>Duration:</strong> ${activity.duration_hours}h</p>
        <p><strong>Time:</strong> ${activity.time_slot}</p>
        ${activity.description ? `<p><strong>Description:</strong> ${activity.description}</p>` : ''}
        <span class="activity-priority">Priority: ${activity.priority}/5</span>
    `;
    
    container.appendChild(card);
}

// Create Plan
async function createPlan() {
    if (!currentAgentId) {
        showMessage('plan-message', 'error', 'Please create a trip plan first');
        return;
    }
    
    const start_date = document.getElementById('start_date').value;
    
    if (!start_date) {
        showMessage('plan-message', 'error', 'Please select a start date');
        return;
    }
    
    const message = document.getElementById('plan-message');
    
    try {
        message.className = 'message';
        message.textContent = 'Creating plan...';
        
        const response = await fetch('/api/create-plan', {
            method: 'POST',
            headers: {
                'Content-Type': 'application/json'
            },
            body: JSON.stringify({
                agent_id: currentAgentId,
                start_date
            })
        });
        
        const data = await response.json();
        
        if (data.success) {
            message.className = 'message success';
            message.textContent = `✓ Plan created! Total cost: $${data.summary.total_cost.toFixed(2)}`;
            
            // Fetch and display itinerary
            await fetchAndDisplayItinerary();
            
            // Show itinerary display
            document.getElementById('itinerary-display').style.display = 'block';
            
            // Scroll to itinerary
            setTimeout(() => {
                document.getElementById('itinerary-display').scrollIntoView({ behavior: 'smooth' });
            }, 300);
        } else {
            message.className = 'message error';
            message.textContent = `✗ Error: ${data.error}`;
        }
    } catch (error) {
        message.className = 'message error';
        message.textContent = `✗ Error: ${error.message}`;
    }
}

// Fetch and Display Itinerary
async function fetchAndDisplayItinerary() {
    try {
        // Get Markdown
        const markdownRes = await fetch(`/api/get-itinerary-markdown?agent_id=${currentAgentId}`);
        const markdownData = await markdownRes.json();
        
        // Get Summary
        const summaryRes = await fetch(`/api/get-summary?agent_id=${currentAgentId}`);
        const summaryDataJson = await summaryRes.json();
        
        // Get JSON
        const jsonRes = await fetch(`/api/get-itinerary-json?agent_id=${currentAgentId}`);
        const jsonData = await jsonRes.json();
        
        if (markdownData.success) {
            document.getElementById('itinerary-markdown').textContent = markdownData.markdown;
        }
        
        if (summaryDataJson.success) {
            const summary = summaryDataJson.summary;
            displaySummary(summary);
        }
        
        if (jsonData.success) {
            document.getElementById('itinerary-json').textContent = jsonData.itinerary;
        }
    } catch (error) {
        console.error('Error fetching itinerary:', error);
    }
}

// Display Summary
function displaySummary(summary) {
    const container = document.getElementById('itinerary-summary');
    
    const summaryItems = [
        { label: 'Days', value: summary.num_days, unit: 'days' },
        { label: 'Activities', value: summary.num_activities, unit: 'total' },
        { label: 'Total Cost', value: `$${summary.total_cost.toFixed(2)}`, unit: '' },
        { label: 'Avg/Day', value: `$${summary.average_cost_per_day.toFixed(2)}`, unit: '/day' },
        { label: 'Total Hours', value: summary.total_hours.toFixed(1), unit: 'hrs' },
        { label: 'Budget Remaining', value: `$${summary.budget_remaining.toFixed(2)}`, unit: '' },
        { label: 'Utilization', value: `${summary.budget_utilization_percent.toFixed(1)}%`, unit: '' }
    ];
    
    container.innerHTML = summaryItems.map(item => `
        <div class="summary-item">
            <div class="summary-item-label">${item.label}</div>
            <div class="summary-item-value">${item.value}</div>
            ${item.unit ? `<div class="summary-item-unit">${item.unit}</div>` : ''}
        </div>
    `).join('');
}

// Generate Scenarios
async function generateScenarios(method) {
    if (!currentAgentId) {
        showMessage('scenarios-message', 'error', 'Please create a plan first');
        return;
    }
    
    try {
        const response = await fetch('/api/generate-scenarios', {
            method: 'POST',
            headers: {
                'Content-Type': 'application/json'
            },
            body: JSON.stringify({
                agent_id: currentAgentId,
                method
            })
        });
        
        const data = await response.json();
        
        if (data.success) {
            document.getElementById('scenarios-comparison').textContent = data.comparison;
            document.getElementById('scenarios-display').style.display = 'block';
            
            setTimeout(() => {
                document.getElementById('scenarios-display').scrollIntoView({ behavior: 'smooth' });
            }, 300);
        } else {
            showMessage('scenarios-message', 'error', `Error: ${data.error}`);
        }
    } catch (error) {
        showMessage('scenarios-message', 'error', `Error: ${error.message}`);
    }
}

// Switch Tabs
function switchTab(tabName) {
    // Hide all tabs
    document.querySelectorAll('.tab-content').forEach(tab => {
        tab.classList.remove('active');
    });
    
    document.querySelectorAll('.tab-btn').forEach(btn => {
        btn.classList.remove('active');
    });
    
    // Show selected tab
    document.getElementById(tabName + '-tab').classList.add('active');
    event.target.classList.add('active');
}

// Show Message
function showMessage(elementId, type, text) {
    const element = document.getElementById(elementId);
    element.className = `message ${type}`;
    element.textContent = text;
}

// Update Activity Type
function updateActivityType() {
    // This can be used to update default values based on activity type
    const type = document.getElementById('activity_type').value;
    
    // Set default values based on type
    const defaults = {
        'accommodation': { cost: 100, duration: 1 },
        'dining': { cost: 30, duration: 2 },
        'sightseeing': { cost: 25, duration: 3 },
        'transportation': { cost: 20, duration: 1 },
        'entertainment': { cost: 50, duration: 3 },
        'activity': { cost: 75, duration: 4 }
    };
    
    if (defaults[type]) {
        document.getElementById('activity_cost').placeholder = `e.g., $${defaults[type].cost}`;
        document.getElementById('activity_duration').placeholder = `e.g., ${defaults[type].duration}h`;
    }
}

// Run Full Demo Workflow
async function runDemoWorkflow() {
    // Step 1: Create agent with demo constraints
    await createAgent();
    
    // Wait a moment then add demo activities
    await sleep(500);
    
    const demoActivities = [
        {
            name: 'Hotel Accommodation (4 nights)',
            type: 'accommodation',
            cost: 500,
            duration: 1,
            time_slot: 'all-day',
            priority: 5,
            description: 'Mid-range hotel in Manhattan'
        },
        {
            name: 'Times Square & Broadway Show',
            type: 'entertainment',
            cost: 150,
            duration: 3,
            time_slot: 'evening',
            priority: 5,
            description: 'Evening Broadway musical'
        },
        {
            name: 'Statue of Liberty Visit',
            type: 'sightseeing',
            cost: 80,
            duration: 4,
            time_slot: 'morning',
            priority: 5,
            description: 'Ferry and pedestal access'
        },
        {
            name: 'Central Park Walking Tour',
            type: 'activity',
            cost: 50,
            duration: 3,
            time_slot: 'afternoon',
            priority: 4,
            description: 'Scenic park walk'
        },
        {
            name: 'Metropolitan Museum',
            type: 'sightseeing',
            cost: 65,
            duration: 4,
            time_slot: 'afternoon',
            priority: 3,
            description: 'Premier art museum'
        },
        {
            name: 'Fine Dining Experience',
            type: 'dining',
            cost: 200,
            duration: 3,
            time_slot: 'evening',
            priority: 2,
            description: 'Premium restaurant'
        }
    ];
    
    // Add all demo activities
    for (let activity of demoActivities) {
        await addDemoActivity(activity);
        await sleep(300);
    }
    
    // Wait then create plan
    await sleep(500);
    await createPlan();
    
    // Wait then generate scenarios
    await sleep(1000);
    await generateScenarios('budget');
}

// Add Demo Activity (helper function)
async function addDemoActivity(activity) {
    if (!currentAgentId) return;
    
    try {
        const response = await fetch('/api/add-activity', {
            method: 'POST',
            headers: {
                'Content-Type': 'application/json'
            },
            body: JSON.stringify({
                agent_id: currentAgentId,
                ...activity
            })
        });
        
        const data = await response.json();
        
        if (data.success) {
            activityCount = data.activity_count;
            addActivityToList(data.activity);
            document.getElementById('activity-count').textContent = activityCount;
        }
    } catch (error) {
        console.error('Error adding demo activity:', error);
    }
}

// Sleep utility function
function sleep(ms) {
    return new Promise(resolve => setTimeout(resolve, ms));
}
```

---

## 4. Flask Backend (app.py)

```python
"""
Flask Web Server for Travel Planner Agent
Provides REST API endpoints for the travel planning system
"""

from flask import Flask, render_template, request, jsonify
import sys
from pathlib import Path

# Add src directory to path
sys.path.insert(0, str(Path(__file__).parent / 'src'))

from constraints import Activity, ActivityType, TravelConstraints
from agent import TravelPlannerAgent

app = Flask(__name__, 
            template_folder='templates',
            static_folder='static')

# Global agents storage (in production, use proper session management)
agents = {}

@app.route('/')
def index():
    """Serve the main web page"""
    return render_template('index.html')

@app.route('/api/activity-types', methods=['GET'])
def get_activity_types():
    """Get available activity types"""
    types = [t.value for t in ActivityType]
    return jsonify({'activity_types': types})

@app.route('/api/create-agent', methods=['POST'])
def create_agent():
    """Create a new travel planner agent with constraints"""
    data = request.json
    
    try:
        constraints = TravelConstraints(
            max_budget=float(data.get('budget', 1000)),
            max_duration_days=int(data.get('duration', 5)),
            min_accommodation_quality=int(data.get('accommodation_quality', 3)),
            must_visit=data.get('must_visit', []),
            avoid_activities=data.get('avoid_activities', []),
            transportation_preference=data.get('transportation', 'mixed')
        )
        
        agent = TravelPlannerAgent(constraints)
        agent_id = f"agent_{len(agents)}"
        agents[agent_id] = agent
        
        return jsonify({
            'success': True,
            'agent_id': agent_id,
            'constraints': constraints.to_dict()
        })
    except Exception as e:
        return jsonify({'success': False, 'error': str(e)}), 400

@app.route('/api/add-activity', methods=['POST'])
def add_activity():
    """Add an activity to the agent"""
    data = request.json
    agent_id = data.get('agent_id')
    
    if agent_id not in agents:
        return jsonify({'success': False, 'error': 'Agent not found'}), 404
    
    try:
        activity = Activity(
            name=data.get('name'),
            type=ActivityType(data.get('type')),
            cost=float(data.get('cost', 0)),
            duration_hours=float(data.get('duration', 1)),
            time_slot=data.get('time_slot', 'morning'),
            description=data.get('description', ''),
            priority=int(data.get('priority', 3))
        )
        
        agent = agents[agent_id]
        success, error = agent.add_activity(activity)
        
        if success:
            return jsonify({
                'success': True,
                'activity': activity.to_dict(),
                'activity_count': len(agent.activity_pool)
            })
        else:
            return jsonify({'success': False, 'error': error}), 400
    except Exception as e:
        return jsonify({'success': False, 'error': str(e)}), 400

@app.route('/api/create-plan', methods=['POST'])
def create_plan():
    """Create an optimized travel plan"""
    data = request.json
    agent_id = data.get('agent_id')
    start_date = data.get('start_date')
    
    if agent_id not in agents:
        return jsonify({'success': False, 'error': 'Agent not found'}), 404
    
    try:
        agent = agents[agent_id]
        success, error = agent.create_optimized_plan(start_date=start_date)
        
        if success:
            summary = agent.get_itinerary_summary()
            return jsonify({
                'success': True,
                'summary': summary,
                'markdown': agent.get_primary_itinerary_markdown()
            })
        else:
            return jsonify({'success': False, 'error': error}), 400
    except Exception as e:
        return jsonify({'success': False, 'error': str(e)}), 400

@app.route('/api/get-itinerary-json', methods=['GET'])
def get_itinerary_json():
    """Get itinerary in JSON format"""
    agent_id = request.args.get('agent_id')
    
    if agent_id not in agents:
        return jsonify({'success': False, 'error': 'Agent not found'}), 404
    
    try:
        agent = agents[agent_id]
        itinerary = agent.get_primary_itinerary_json()
        return jsonify({
            'success': True,
            'itinerary': itinerary
        })
    except Exception as e:
        return jsonify({'success': False, 'error': str(e)}), 400

@app.route('/api/get-itinerary-markdown', methods=['GET'])
def get_itinerary_markdown():
    """Get itinerary in Markdown format"""
    agent_id = request.args.get('agent_id')
    
    if agent_id not in agents:
        return jsonify({'success': False, 'error': 'Agent not found'}), 404
    
    try:
        agent = agents[agent_id]
        markdown = agent.get_primary_itinerary_markdown()
        return jsonify({
            'success': True,
            'markdown': markdown
        })
    except Exception as e:
        return jsonify({'success': False, 'error': str(e)}), 400

@app.route('/api/get-summary', methods=['GET'])
def get_summary():
    """Get itinerary summary statistics"""
    agent_id = request.args.get('agent_id')
    
    if agent_id not in agents:
        return jsonify({'success': False, 'error': 'Agent not found'}), 404
    
    try:
        agent = agents[agent_id]
        summary = agent.get_itinerary_summary()
        return jsonify({
            'success': True,
            'summary': summary
        })
    except Exception as e:
        return jsonify({'success': False, 'error': str(e)}), 400

@app.route('/api/generate-scenarios', methods=['POST'])
def generate_scenarios():
    """Generate alternative scenarios"""
    data = request.json
    agent_id = data.get('agent_id')
    method = data.get('method', 'budget')
    
    if agent_id not in agents:
        return jsonify({'success': False, 'error': 'Agent not found'}), 404
    
    try:
        agent = agents[agent_id]
        success, scenarios = agent.generate_scenarios(method=method)
        
        if success:
            # Convert scenarios to JSON-serializable format
            serialized_scenarios = {}
            for key, scenario in scenarios.items():
                scenario_copy = scenario.copy()
                if 'activities' in scenario_copy:
                    scenario_copy['activities'] = [a.to_dict() for a in scenario_copy['activities']]
                serialized_scenarios[key] = scenario_copy
            
            return jsonify({
                'success': True,
                'method': method,
                'scenarios': serialized_scenarios,
                'comparison': agent.get_scenario_comparison()
            })
        else:
            return jsonify({'success': False, 'error': 'Failed to generate scenarios'}), 400
    except Exception as e:
        return jsonify({'success': False, 'error': str(e)}), 400

@app.route('/api/agent-status', methods=['GET'])
def agent_status():
    """Get current agent status"""
    agent_id = request.args.get('agent_id')
    
    if agent_id not in agents:
        return jsonify({'success': False, 'error': 'Agent not found'}), 404
    
    try:
        agent = agents[agent_id]
        status = agent.get_agent_status()
        return jsonify({
            'success': True,
            'status': status
        })
    except Exception as e:
        return jsonify({'success': False, 'error': str(e)}), 400

if __name__ == '__main__':
    print("=" * 80)
    print("Travel Planner Web Server")
    print("=" * 80)
    print("\nStarting Flask development server...")
    print("Open your browser to: http://localhost:5000")
    print("\nPress Ctrl+C to stop the server")
    print("=" * 80 + "\n")
    
    app.run(debug=True, port=5000)
```

---

## Installation & Setup

### Prerequisites
- Python 3.8+
- Flask 3.1.2
- Werkzeug 3.1.3

### Install Dependencies
```bash
pip install flask==3.1.2 werkzeug==3.1.3
```

### Run the Server
```bash
python app.py
```

Then open your browser to: `http://localhost:5000`

---

## Features

✅ Step-by-step trip planning workflow
✅ Budget constraint management
✅ Activity prioritization (1-5 scale)
✅ Multiple output formats (Markdown, JSON, Summary)
✅ Alternative scenario generation (budget, priority, cost-reduction)
✅ Real-time budget tracking
✅ Responsive design (mobile-friendly)
✅ One-click demo workflow
✅ Beautiful UI with gradient backgrounds

---

Ready for GitHub! 🚀

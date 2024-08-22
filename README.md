# Vaccout
Dependencies
{
  "name": "vacation-finder",
  "version": "1.0.0",
  "dependencies": {
    "react": "^18.2.0",
    "react-dom": "^18.2.0",
    "react-router-dom": "^6.8.1",
    "axios": "^1.2.3"
  }
}

index.js
import React from 'react';
import ReactDOM from 'react-dom';
import { BrowserRouter as Router } from 'react-router-dom';
import App from './App';

ReactDOM.render(
  <Router>
    <App />
  </Router>,
  document.getElementById('root')
);


App.js

import React from 'react';
import { Routes, Route } from 'react-router-dom';
import HomePage from './pages/HomePage';
import LoginPage from './pages/LoginPage';
import ProfilePage from './pages/ProfilePage';

function App() {
  return (
    <div className="App">
      <Routes>
        <Route path="/" element={<HomePage />} />
        <Route path="/login" element={<LoginPage />} />
        <Route path="/profile" element={<ProfilePage />} />
      </Routes>
    </div>
  );
}

export default App;


SearchBar.js
import React, { useState } from 'react';
import axios from 'axios';

const SearchBar = () => {
  const [query, setQuery] = useState('');
  const [results, setResults] = useState([]);

  const handleSearch = async () => {
    try {
      const response = await axios.get(`/api/search?query=${query}`);
      setResults(response.data);
    } catch (error) {
      console.error('Error fetching search results:', error);
    }
  };

  return (
    <div>
      <input
        type="text"
        value={query}
        onChange={(e) => setQuery(e.target.value)}
      />
      <button onClick={handleSearch}>Search</button>
      <ul>
        {results.map((result) => (
          <li key={result.id}>{result.name}</li>
        ))}
      </ul>
    </div>
  );
};

export default SearchBar;

Recommendations.js
import React, { useEffect, useState } from 'react';
import axios from 'axios';

const Recommendations = () => {
  const [recommendations, setRecommendations] = useState([]);

  useEffect(() => {
    const fetchRecommendations = async () => {
      try {
        const response = await axios.get('/api/recommendations');
        setRecommendations(response.data);
      } catch (error) {
        console.error('Error fetching recommendations:', error);
      }
    };

    fetchRecommendations();
  }, []);

  return (
    <div>
      <h2>Recommendations</h2>
      <ul>
        {recommendations.map((rec) => (
          <li key={rec.id}>{rec.name}</li>
        ))}
      </ul>
    </div>
  );
};

export default Recommendations;


Backend Folder Structure
/backend
  /controllers
    searchController.js
    authController.js
    recommendationsController.js
  /routes
    searchRoutes.js
    authRoutes.js
    recommendationsRoutes.js
  /models
    userModel.js
    destinationModel.js
  app.js
  package.json

package.json (Dependencies for Node.js and Express):
{
  "name": "vacation-finder-backend",
  "version": "1.0.0",
  "dependencies": {
    "express": "^4.18.2",
    "mongoose": "^6.8.0",
    "cors": "^2.8.5",
    "dotenv": "^16.0.3",
    "axios": "^1.2.3"
  }
}

app.js
const express = require('express');
const mongoose = require('mongoose');
const cors = require('cors');
const dotenv = require('dotenv');

dotenv.config();

const app = express();
app.use(cors());
app.use(express.json());

const searchRoutes = require('./routes/searchRoutes');
const authRoutes = require('./routes/authRoutes');
const recommendationsRoutes = require('./routes/recommendationsRoutes');

app.use('/api/search', searchRoutes);
app.use('/api/auth', authRoutes);
app.use('/api/recommendations', recommendationsRoutes);

mongoose.connect(process.env.MONGO_URI, {
  useNewUrlParser: true,
  useUnifiedTopology: true,
});

app.listen(process.env.PORT, () => {
  console.log(`Server running on port ${process.env.PORT}`);
});


searchController.js
const axios = require('axios');

exports.searchDestinations = async (req, res) => {
  const query = req.query.query;
  // Example call to a travel API (e.g., OpenTripMap)
  try {
    const response = await axios.get(`https://api.opentripmap.com/0.1/en/places/autocomplete?name=${query}&apikey=${process.env.TRAVEL_API_KEY}`);
    res.json(response.data);
  } catch (error) {
    res.status(500).send('Error fetching search results');
  }
};


searchRoutes.js
const express = require('express');
const router = express.Router();
const searchController = require('../controllers/searchController');

router.get('/', searchController.searchDestinations);

module.exports = router;


** Database Schema (Using Mongoose for MongoDB)**
**userModel.js**
const mongoose = require('mongoose');

const userSchema = new mongoose.Schema({
  username: { type: String, required: true },
  email: { type: String, required: true, unique: true },
  password: { type: String, required: true },
});

const User = mongoose.model('User', userSchema);

module.exports = User;

**destinationModel.js**
const mongoose = require('mongoose');

const destinationSchema = new mongoose.Schema({
  name: { type: String, required: true },
  description: String,
  location: {
    type: { type: String, default: 'Point' },
    coordinates: [Number],
  },
});

const Destination = mongoose.model('Destination', destinationSchema);

module.exports = Destination;


**Generative AI Integration (Example with OpenAI GPT-4)**
**recommendationsController.js:**

const axios = require('axios');

exports.getRecommendations = async (req, res) => {
  const preferences = req.query.preferences; // Example: "beach, adventure"
  try {
    const response = await axios.post('https://api.openai.com/v1/engines/gpt-4/completions', {
      prompt: `Provide vacation recommendations based on these preferences: ${preferences}`,
      max_tokens: 150,
    }, {
      headers: {
        'Authorization': `Bearer ${process.env.OPENAI_API_KEY}`,
      },
    });
    res.json(response.data.choices[0].text);
  } catch (error) {
    res.status(500).send('Error fetching recommendations');
  }
};

**recommendationsRoutes.js**
const express = require('express');
const router = express.Router();
const recommendationsController = require('../controllers/recommendationsController');

router.get('/', recommendationsController.getRecommendations);

module.exports = router;





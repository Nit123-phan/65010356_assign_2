const express = require('express');
const axios = require('axios');
const app = express();
const port = 3000;

app.use(express.json());

app.get('/', (req, res) => {
  res.send('Drone API Server');
});

app.listen(port, () => {
  console.log(`Server is running on http://localhost:${port}`);
});

app.get('/configs/:id', async (req, res) => {
  const droneId = parseInt(req.params.id);
  console.log('Requested drone ID:', droneId); 

  try {
    const response = await axios.get(`https://script.googleusercontent.com/macros/echo?user_content_key=IobQnizvZKiLnsq9Qzwa04bbPcw8HP9CB67cy1k2jVsXgh6K6LcwHhfgtK5kzCJ6pZCeXwK9KLCVgFKpDPebk2rXRkDdwsTHm5_BxDlH2jW0nuo2oDemN9CCS2h10ox_1xSncGQajx_ryfhECjZEnOQwROx_Wq-O5wsPy5w5JUsdPdcpj8TWgjjVAuN4sDTiMrnThHKU7n7LmNcslGllO5_ldGegmAJuXjfvqC1tFaecv-CYmXuM6Nz9Jw9Md8uu&lib=M9_yccKOaZVEQaYjEvK1gClQlFAuFWsxN`);
    const data = response.data;

    console.log('All drone data:', data); 
    const drone = data.data.find(d => d.drone_id === droneId);

    if (!drone) {
      return res.status(404).send('Drone not found');
    }

    const filteredData = {
      drone_id: drone.drone_id,
      drone_name: drone.drone_name,
      light: drone.light,
      condition: drone.condition,
      max_speed: drone.max_speed,
    };

    // เงื่อนไข max_speed 
    if (!filteredData.max_speed) {
      filteredData.max_speed = 100;
    } else if (filteredData.max_speed > 110) {
      filteredData.max_speed = 110;
    }

    console.log('Filtered drone data:', filteredData);

    res.json(filteredData); 
  } catch (error) {
    console.error('Error details:', error.message);
    res.status(500).send('Error fetching drone config');
  }
});

app.get('/status/:id', (req, res) => {
    const droneId = req.params.id;
    res.json({ condition: 'good' });
});

app.get('/logs', async (req, res) => {
    try {
      const response = await axios.get(`https://app-tracking.pockethost.io/api/collections/drone_logs/records`);

      const logs = response.data;
      res.json(logs);
    } catch (error) {
      res.status(500).send('Error fetching logs');
    }
});

app.post('/logs', async (req, res) => {
    const newLog = req.body;
    try {
      const response = await axios.post(`https://app-tracking.pockethost.io/api/collections/drone_logs/records`,req.body,{
        headers:{
          'Content-Type' : 'application/json',
        },
      });

      res.json(response.data);
    } catch (error) {
      res.status(500).send('Error posting log');
    }
});
  
  
  
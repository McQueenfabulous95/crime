# crime
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Crime Mapping</title>
    <!-- Leaflet CSS -->
    <link rel="stylesheet" href="https://unpkg.com/leaflet/dist/leaflet.css" />
    <!-- Leaflet MarkerCluster CSS -->
    <link rel="stylesheet" href="https://unpkg.com/leaflet.markercluster/dist/MarkerCluster.css" />
    <link rel="stylesheet" href="https://unpkg.com/leaflet.markercluster/dist/MarkerCluster.Default.css" />
    <!-- Heatmap.js CSS -->
    <link rel="stylesheet" href="https://unpkg.com/leaflet-heat/dist/leaflet-heat.css" />
    <style>
        #map {
            height: 500px;
        }
    </style>
</head>
<h>crime mapping</h>
<body>
    <div id="map"></div>
    <button onclick="toggleHeatmap()">Toggle Heatmap</button>
    <button onclick="getCrimeCount()">Get Crime Count</button>

    <!-- Leaflet JavaScript -->
    <script src="https://unpkg.com/leaflet/dist/leaflet.js"></script>
    <!-- Leaflet MarkerCluster JavaScript -->
    <script src="https://unpkg.com/leaflet.markercluster/dist/leaflet.markercluster.js"></script>
    <!-- Heatmap.js JavaScript -->
    <script src="https://unpkg.com/leaflet-heat/dist/leaflet-heat.js"></script>

    <script>
        var map = L.map('map').setView([40.7128, -74.0060], 12);
        var markerGroup; // Variable to store marker group

        // OpenStreetMap tiles
        L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
            attribution: '&copy; OpenStreetMap contributors'
        }).addTo(map);

        // Fetch random crime data from PHP script
        fetch('get_random_crime_data.php')
            .then(response => response.json())
            .then(data => {
                // Process data and add markers to the map
                var markers = [];
                var heatData = [];

                data.forEach(crime => {
                    // Create markers with different icons for each crime type
                    var icon = getCrimeIcon(crime.type);
                    var marker = L.marker([parseFloat(crime.latitude), parseFloat(crime.longitude)], { icon: icon })
                        .bindPopup(`Type: ${crime.type}<br>Intensity: ${crime.intensity}`);
                    
                    markers.push(marker);

                    // Add data points for heatmap
                    heatData.push([parseFloat(crime.latitude), parseFloat(crime.longitude), crime.intensity]);
                });

                // Add markers to a marker cluster group
                markerGroup = L.markerClusterGroup();
                markerGroup.addLayers(markers);
                map.addLayer(markerGroup);

                // Create heatmap layer based on crime data
                var heatLayer = L.heatLayer(heatData, { radius: 20 });

                // Toggle heatmap layer visibility
                window.toggleHeatmap = function() {
                    if (map.hasLayer(heatLayer)) {
                        map.removeLayer(heatLayer);
                    } else {
                        map.addLayer(heatLayer);
                    }
                };

                // Function to get crime count in the current view
                window.getCrimeCount = function() {
                    var bounds = map.getBounds();
                    var count = 0;

                    data.forEach(crime => {
                        var crimeLatLng = L.latLng(parseFloat(crime.latitude), parseFloat(crime.longitude));

                        if (bounds.contains(crimeLatLng)) {
                            count++;
                        }
                    });

                    alert(`Number of crimes in the current view: ${count}`);
                };
            })
            .catch(error => console.error('Error:', error));

        // Define marker icons for different crime types
        function getCrimeIcon(crimeType) {
            var icons = {
                theft: L.icon({ iconUrl: 'https://leafletjs.com/examples/custom-icons/theft.png', iconSize: [32, 32] }),
                assault: L.icon({ iconUrl: 'https://leafletjs.com/examples/custom-icons/assault.png', iconSize: [32, 32] }),
                vandalism: L.icon({ iconUrl: 'https://leafletjs.com/examples/custom-icons/vandalism.png', iconSize: [32, 32] }),
                // Add more icons for other crime types
            };

            return icons[crimeType] || L.icon({ iconUrl: 'https://leafletjs.com/examples/custom-icons/default.png', iconSize: [32, 32] });
        }
    </script>
</body>
</html>

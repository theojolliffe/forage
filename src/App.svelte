<script>
  import { onMount } from "svelte";
  import L from "leaflet";
  
  let map, data, uniqueArray, redIcon, blueIcon;

  onMount(async () => {
    const response = await fetch("/records.json");
    data = await response.json();
    uniqueArray = [...new Set(data.map(e => e['Common Name']))];
    uniqueArray = uniqueArray.filter(e => e != "NaN")

    console.log("data", data)

    map = L.map('map').setView([51.54, -0.15], 13);
  
    L.tileLayer('https://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png', {
      attribution: '&copy; <a href="https://www.openstreetmap.org/copyright">OpenStreetMap</a> contributors'
    }).addTo(map);

    redIcon = new L.Icon({
        iconUrl: 'https://raw.githubusercontent.com/pointhi/leaflet-color-markers/master/img/marker-icon-2x-red.png',
        shadowUrl: 'https://cdnjs.cloudflare.com/ajax/libs/leaflet/0.7.7/images/marker-shadow.png',
        iconSize: [25, 41],
        iconAnchor: [12, 41],
        popupAnchor: [1, -34],
        shadowSize: [41, 41]
    });
    blueIcon = new L.Icon({
        iconUrl: 'https://raw.githubusercontent.com/pointhi/leaflet-color-markers/master/img/marker-icon-2x-blue.png',
        shadowUrl: 'https://cdnjs.cloudflare.com/ajax/libs/leaflet/0.7.7/images/marker-shadow.png',
        iconSize: [25, 41],
        iconAnchor: [12, 41],
        popupAnchor: [1, -34],
        shadowSize: [41, 41]
    });
  });

  let markers = [];

  function clearMarkers() {

    console.log("clear");
    markers.forEach(marker => {
        map.removeLayer(marker);
    });
    markers = [];
  }

  function filterData(treeSuggestion) {
    console.log("treeSuggestion", treeSuggestion);

    let data_temp = data.filter(e => e['Common Name'].toLowerCase()==treeSuggestion.toLowerCase() )
    clearMarkers();

    data_temp.forEach((tree, i) => {

      if ((!isNaN(tree.Latitude)) & (!isNaN(tree.Longitude))) {

        const markerLocation = [tree.Latitude, tree.Longitude];
        const marker = L.marker(markerLocation, { icon: redIcon }).addTo(map);

        marker.bindPopup(`
            <h4 id="restaurant-${i}">
                ${tree['Common Name']}
            </h4>
            <p>
              ${tree['Location']}
            </p>
            <p>
              ${tree['Scientific Name']}
            </p>
        `);
        markers.push(marker);
      };
    
    });

  }

  let searchName = '';
  let uniqueArrayFiltered;

  $: if (searchName) {
    uniqueArrayFiltered = uniqueArray.filter(e => e.toLowerCase().includes(searchName.toLowerCase()) )
  }

</script>

<div id="map" style="height: 700px; width: 100%;"></div>

<br>
<input bind:value={searchName} placeholder="Search a tree" />
{#if searchName}
  {#each uniqueArrayFiltered as treeSuggestion}
    <button on:click={() => filterData(treeSuggestion)}>{treeSuggestion}</button>
  {/each}
{/if}
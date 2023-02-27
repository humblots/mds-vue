<script setup lang="ts"></script>
<script lang="ts">
import { autofill } from "@mapbox/search-js-web";
import mapboxgl from "mapbox-gl";
import { toRaw } from "vue";

export default {
  data() {
    return {
      firstname: "Jean",
      age: 30,
      year: 2022,
      buttonVisible: false,
      family: [
        {
          firstname: "Jojo",
          lastname: "Bernard",
          age: 25,
          picture:
            "https://cdn0.tnwcdn.com/wp-content/blogs.dir/1/files/2019/02/pdne3.png",
          phoneNumber: "06 01 02 03 04",
          fullAddress: {
            address: "2 Rue Marc Leroux",
            coordinates: [6.126069111269203, 45.91526403633885],
            marker: null,
          },
        },
        {
          firstname: "Marie",
          lastname: "Blachère",
          age: 29,
          picture:
            "https://cdn0.tnwcdn.com/wp-content/blogs.dir/1/files/2019/02/pdne3.png",
          phoneNumber: "06 02 03 04 05",
          fullAddress: {
            address: "2 Avenue du Rhône",
            coordinates: [6.12082414010444, 45.897790945241695],
            marker: null,
          },
        },
        {
          firstname: "Jean",
          lastname: "Bernard",
          age: 20,
          picture:
            "https://cdn0.tnwcdn.com/wp-content/blogs.dir/1/files/2019/02/pdne3.png",
          phoneNumber: "06 03 04 05 06",
          fullAddress: {
            address: "2 Chemin de Bellevue",
            coordinates: [6.157646561807065, 45.91849523570724],
            marker: null,
          },
        },
        {
          firstname: "Paul",
          lastname: "Pogba",
          age: 17,
          picture:
            "https://cdn0.tnwcdn.com/wp-content/blogs.dir/1/files/2019/02/pdne3.png",
          phoneNumber: "06 04 05 06 07",
          fullAddress: {
            address: "2 Passage de la cathédrale",
            coordinates: [6.125466838443138, 45.89956442089171],
            marker: null,
          },
        },
      ],
      formValues: {
        firstname: "",
        lastname: "",
        age: "",
        phoneNumber: "",
        address: "",
        errors: [],
      },
      map: null,
      geojson: null,
      token:
        "pk.eyJ1IjoiaHVtYmxvdHMiLCJhIjoiY2xlNzRjZ3d1MDF3MzN3bWVkdHdydWFzcCJ9.Blgw7RI3qBnpucybMeLb6Q",
    };
  },
  methods: {
    increment() {
      setInterval(() => {
        this.age++;
        this.year++;
      }, 3000);
    },
    updateFirstname() {
      this.firstname = this.firstname === "Jean" ? "Patrick" : "Jean";
    },
    addMember(member: any) {
      this.family.push(member);
    },
    removeMember(i: number) {
      this.family[i].fullAddress.marker.remove();
      this.family.splice(i, 1);
    },
    async checkForm(e: any): any {
      e.preventDefault();
      this.formValues.errors = [];

      const { age, firstname, lastname, phoneNumber, address } =
        this.formValues;
      if (
        age === "" ||
        firstname === "" ||
        lastname === "" ||
        phoneNumber === "" ||
        address === ""
      ) {
        return this.formValues.errors.push("Missing values");
      }
      const regexp = new RegExp(
        /^[+]?[(]?[0-9]{3}[)]?[-\s.]?[0-9]{3}[-\s.]?[0-9]{4,6}$/
      );
      if (phoneNumber.match(regexp) === null) {
        return this.formValues.errors.push("incorrect phone number");
      }
      const coordinates = this.geojson.features[0].geometry.coordinates;
      const fullAddress = {
        address,
        coordinates: coordinates,
        marker: this.createMarker(coordinates),
      };
      const member = { age, firstname, lastname, fullAddress, phoneNumber };
      this.addMember(member);
      this.addMarkerToMap(member);
    },
    createMarker(coordinates) {
      return new mapboxgl.Marker({ color: "green" }).setLngLat(
        toRaw(coordinates)
      );
    },
    addMarkerToMap(member) {
      const { firstname, lastname, age, phoneNumber, picture, fullAddress } =
        member;
      const marker = fullAddress.marker;
      // make a marker for each feature and add to the map
      const popup = new mapboxgl.Popup({ offset: 25 }).setText(
        `${firstname} ${lastname} - ${age} years old`
      );
      marker.setPopup(popup).addTo(this.map);

      const markerDiv = marker.getElement();
      markerDiv.addEventListener("mouseenter", () => {
        const infosDiv = document.getElementById("member-infos");
        infosDiv.innerHTML = `<h1>${firstname} ${lastname}</h1>
            <img class="family-img" src="${picture}" />
            <p>Age: ${age}</p>
            <p>Phone: ${phoneNumber}</p>
            <p>Address: ${fullAddress.address} </p>`;
        marker.togglePopup();
      });
      markerDiv.addEventListener("mouseleave", () => {
        marker.togglePopup();
        const infosDiv = document.getElementById("member-infos");
        infosDiv.innerHTML = "";
      });

      return marker;
    },
    flyToRandomMarker() {
      if (!this.family.length) return;
      const coordinates =
        this.family[Math.floor(Math.random() * this.family.length)].fullAddress
          .coordinates;
      this.map.flyTo({ center: coordinates, zoom: 20 });
    },
  },
  computed: {
    orderedByAgeFamily(): Array<any> {
      return this.family.sort((a, b) => a.age > b.age);
    },
  },
  mounted() {
    this.increment();
    const autofiller = autofill({
      accessToken: this.token,
    });

    autofiller.addEventListener("retrieve", (event) => {
      this.geojson = event.detail;
    });

    this.map = new mapboxgl.Map({
      container: "map", // container ID
      accessToken: this.token,
      style: "mapbox://styles/mapbox/dark-v11", // style URL
      center: [6.116990138253653, 45.89986037274524], // starting position [lng, lat]
      zoom: 12, // starting zoom
    });

    this.family.forEach((member: any) => {
      const { firstname, age, fullAddress } = member;
      member.fullAddress.marker = this.createMarker(fullAddress.coordinates);
      this.addMarkerToMap(member);
    });
  },
};
</script>

<template>
  <main>
    <div id="presentation">
      <p>Bonjour, je m'appelle {{ firstname }}</p>
    </div>
    <div id="age">
      <p>En {{ year }}, {{ firstname }} aura {{ age }} ans</p>
    </div>
    <div id="family">
      <div
        id="family-member"
        v-bind:key="member.firstname"
        v-for="(member, i) in orderedByAgeFamily"
      >
        <p>
          Nom: {{ member.lastname }} - Prenom: {{ member.firstname }} - Age:
          {{ member.age }} - Tel: {{ member.phoneNumber }} - Adresse:
          {{ member.fullAddress.address }}
        </p>
        <button @click="() => removeMember(i)">Remove</button>
      </div>
    </div>
    <button v-if="buttonVisible" @click="updateFirstname" id="crazy-button">
      Crazy Button
    </button>

    <form id="family-form" @submit="checkForm">
      <div v-if="this.formValues.errors.length">
        <b>Please correct the following error(s):</b>
        <ul>
          <li v-bind:key="error" v-for="error in this.formValues.errors">
            {{ error }}
          </li>
        </ul>
      </div>

      <div class="form-field">
        <label for="name">Last Name</label>
        <input type="text" name="lastname" v-model="formValues.lastname" />
      </div>
      <div class="form-field">
        <label for="name">First Name</label>
        <input type="text" name="firstname" v-model="formValues.firstname" />
      </div>
      <div class="form-field">
        <label for="age">Age</label>
        <input type="number" name="age" v-model="formValues.age" min="10" />
      </div>
      <div class="form-field">
        <label for="name">Phone Number</label>
        <input type="tel" name="phoneNumber" v-model="formValues.phoneNumber" />
      </div>
      <div class="form-field">
        <label for="name">Address</label>
        <input
          type="text"
          name="address"
          v-model="formValues.address"
          autocomplete="street-address"
        />
      </div>
      <button type="submit" @submit="addMember" id="add-family-member">
        Add member
      </button>
    </form>
    <div class="map-container">
      <div id="map"></div>
      <div class="map-container-right">
        <div id="member-infos"></div>
        <button id="fly-btn" @click="flyToRandomMarker">
          Fly to random marker
        </button>
      </div>
    </div>
  </main>
</template>

<style>
main {
  min-height: 100vh;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
}

#family-member {
  display: flex;
  gap: 2rem;
  margin-bottom: 10px;
}

#family-form {
  display: flex;
  flex-direction: column;
  gap: 1rem;
  margin-bottom: 10px;
}

.form-field {
  display: flex;
  justify-content: space-between;
}

.form-field label {
  margin-right: 20px;
}

.map-container {
  display: flex;
  flex-direction: row;
}

#fly-btn {
  height: fit-content;
}

#map {
  width: 600px;
  height: 400px;
}

.map-container-right {
  width: 200px;
}

.family-img {
  max-width: 100px;
  max-height: 150px;
}

.mapboxgl-popup-content {
  color: black;
  max-width: 400px;
  font: 12px/20px "Helvetica Neue", Arial, Helvetica, sans-serif;
}

.mapboxgl-popup-close-button {
  display: none;
}
</style>

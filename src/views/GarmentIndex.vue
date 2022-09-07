<script>
import axios from "axios";
export default {
  data: function () {
    return {
      garments: [],
      colors: [],
      categories: [],
      currentGarment: {},
      nameSearch: "",
      garmentName: "",
      garmentImage: "",
      garmentImageFile: "",
      garmentColorId: "",
      garmentCategoryId: "",
    };
  },
  created: function () {
    this.indexGarments();
  },
  watch: {
    currentGarment: function () {
      console.log(this.currentGarment);
    },
  },
  methods: {
    indexGarments: function () {
      axios.get("/garments.json").then((response) => {
        this.garments = response.data.garments;
        this.colors = response.data.colors;
        this.categories = response.data.categories;
        console.log("Wardrobe: ", this.garments, this.colors, this.categories);
      });
    },
    filterGarments() {
      return this.garments.filter((garment) => {
        var lowerName = garment.name.toLowerCase();
        var lowerNameSearch = this.nameSearch.toLowerCase();
        return lowerName.includes(lowerNameSearch);
      });
    },
    destroyGarment: function (garment) {
      axios.delete("/garments/" + garment.id + ".json").then((response) => {
        console.log(response.data);
        var index = this.garments.indexOf(garment);
        this.garments.splice(index, 1);
      });
    },
    setFile: function (event) {
      if (event.target.files.length > 0) {
        this.garmentImageFile = event.target.files[0];
      }
    },
    createGarment: function () {
      console.log("hello");
      var formData = new FormData();
      formData.append("name", this.garmentName);
      formData.append("color_id", this.garmentColorId);
      formData.append("image_file", this.garmentImageFile);
      formData.append("category_id", this.garmentCategoryId);
      formData.append("image", this.garmentImage);

      axios.post("/garments", formData).then((response) => {
        this.garmentName = "";
        this.garmentImage = "";
        this.garmentCategoryId = "";
        this.garmentColorId = "";
        this.garmentImageFile = "";
        console.log(response.data);
        var modal = document.getElementById("#exampleModalCenter");
        modal.style.display = "none";
        this.indexGarments();
      });
    },
  },
};
</script>

<template>
  <div>
    <h1>Wardrobe</h1>
    <div class="row">
      <div class="row p-3">
        Search:
        <input type="search" v-model="nameSearch" list="names" />
        <datalist id="names">
          <option v-for="garment in filterGarments()" v-bind:key="garment.id">{{ garment.name }}</option>
        </datalist>
      </div>
      <button type="button" class="btn btn-primary" data-toggle="modal" data-target="#exampleModalCenter">
        Add New Item
      </button>
      <!-- Modal -->
      <div
        class="modal fade"
        id="exampleModalCenter"
        tabindex="-1"
        role="dialog"
        aria-labelledby="exampleModalCenterTitle"
        aria-hidden="true"
      >
        <div class="modal-dialog modal-dialog-centered" role="document">
          <div class="modal-content">
            <div class="modal-header">
              <h5 class="modal-title" id="exampleModalLongTitle">Add new item</h5>
              <button type="button" class="close" data-dismiss="modal" aria-label="Close">
                <span aria-hidden="true">&times;</span>
              </button>
            </div>
            <div class="modal-body">
              <form v-on:submit.prevent="createGarment()">
                <ul>
                  <li v-for="error in errors" v-bind:key="error">
                    {{ error }}
                  </li>
                </ul>
                <div>
                  <div class="input-group input-group-sm mb-3">
                    <div class="input-group-prepend">
                      <span class="input-group-text" id="Name">Name</span>
                    </div>
                    <input
                      type="text"
                      class="form-control"
                      aria-label="Small"
                      aria-describedby="Name"
                      v-model="garmentName"
                    />
                  </div>
                </div>
                <div>
                  <div class="input-group input-group-sm mb-3">
                    <div class="input-group-prepend">
                      <span class="input-group-text" id="ImageUrl">Image Url</span>
                    </div>
                    <input
                      type="text"
                      class="form-control"
                      aria-label="Small"
                      aria-describedby="ImageUrl"
                      v-model="garmentImage"
                    />
                  </div>
                </div>
                <div class="input-group mb-3">
                  <div class="input-group-prepend">
                    <span class="input-group-text">Upload Image</span>
                  </div>
                  <div class="custom-file">
                    <input v-on:change="setFile($event)" type="file" class="custom-file-input" id="inputGroupFile01" />
                    <label class="custom-file-label" for="inputGroupFile01">Choose file</label>
                  </div>
                </div>
                <div>Select Color: {{ selected }}</div>

                <select v-model="garmentColorId">
                  <option disabled value="">Please select one</option>
                  <option v-for="color in colors" v-bind:key="color.id" :value="color.id">{{ color.name }}</option>
                </select>
                <select v-model="garmentCategoryId">
                  <option disabled value="">Please select one</option>
                  <option v-for="category in categories" v-bind:key="category.id" :value="category.id">
                    {{ category.name }}
                  </option>
                </select>
                <input type="submit" class="btn btn-primary" value="Submit Item" />
              </form>
            </div>
          </div>
        </div>
      </div>
      <TransitionGroup name="list">
        <div
          class="col-sm-4"
          v-for="garment in filterGarments()"
          v-bind:key="garment.id"
          v-on:mouseover="currentGarment = garment"
        >
          <div class="card mb-4" v-bind:class="{ selected: garment === currentGarment }">
            <img class="m-auto card-img-top" v-bind:src="garment.image" v-bind:alt="garment.name" />
            <div class="card-body">
              <h5 class="card-title">{{ garment.name }}</h5>
              <!-- <a v-bind:href="`/garments/${garment.id}`" class="btn btn-primary">More Info</a> -->
              <button v-on:click="destroyGarment(garment)">Delete</button>
            </div>
          </div>
        </div>
      </TransitionGroup>
    </div>
  </div>
</template>

<style scoped>
.card {
  background-color: MediumSeaGreen;
}
.card img {
  object-fit: cover;
  height: 250px;
}
.selected {
  /* color: white; */
  background-color: MediumSeaGreen;
  transition: background-color 1s ease;
}
.list-move,
.list-enter-active,
.list-leave-active {
  transition: all 0.5s ease;
}
.list-enter-from,
.list-leave-to {
  opacity: 0;
  transform: translateX(30px);
}
.list-leave-active {
  position: absolute;
}
</style>

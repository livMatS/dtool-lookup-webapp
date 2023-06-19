<template>
  <div id="app" class="container-fluid">
    <header>
  <div v-if="!token">
    <nav class="navbar navbar-dark bg-dark p-2 w-100">
      <span class="navbar-brand mb-0 h1">dtool</span>
    </nav>
  </div>
  <div v-else>
    <nav class="navbar navbar-dark bg-dark p-2">
      <span class="navbar-brand mb-0 h1 mr-auto">dtool</span>
      <div class="d-flex align-items-center justify-content-end">
        <TextSearch @start-search="searchDatasets" />
      </div>
      <div>
        <b-button pill variant="outline-danger" @click="logout()">Logout</b-button>
      </div>
    </nav>
  </div>
</header>

    <div v-if="token" class="">
      <div class="row row-height">
        <div class="col-md-2 overflow-auto h-100 pr-0">
          <SummaryInfo
            :auth_str="auth_str"
            :lookup_url="lookup_url"
            @start-search="searchDatasets"
          />
        </div>
        <div class="col-md-4 overflow-auto h-100 p-0">
          <div v-if="searchLoading" class="spinner-border text-primary">
            <span class="sr-only">Loading...</span>
          </div>

          <div v-else>
            <div v-if="searchErrored">
              <p>Unable to load datasets please try again.</p>
              <a
                href=""
                class="btn btn-secondary"
                @click.prevent="searchDatasets()"
                >Try again</a
              >
              <p>Or try logging out and in again.</p>
              <a href="" class="btn btn-secondary" @click.prevent="logout()"
                >Logout</a
              >
            </div>

            <div v-else>
              <DatasetTable
                :datasetHits="datasetHits"
                :responseheaders="responseheaders"
                @update-dataset="updateDataset"
              />
              <div v-if="shouldShowPagination">
                <b-pagination
                  v-model="pageNumber"
                  :total-rows="pagination.total"
                  :per-page="this.$store.state.update_current_Per_Page"
                  first-text="First"
                  prev-text="Prev"
                  next-text="Next"
                  last-text="Last"
                  @click="searchDatasets"
                  class="paginationcomponent"
                ></b-pagination>
              </div>
            </div>
          </div>
        </div>

        <div v-if="datasetLoaded" class="col-md-6 overflow-auto h-100 pl-0">
          <div class="card">
            <div class="card-header">
              <div v-if="manifestLoading" class="text-primary">
                <span class="sr-only">Loading...</span>
              </div>
              <div v-else>
                <div v-if="manifestErrored">
                  <p>Unable to load manifest please try again.</p>
                  <a
                    href=""
                    class="btn btn-secondary"
                    @click.prevent="updateManifest()"
                    >Try again</a
                  >
                  <p>Or try logging out and in again.</p>
                  <a href="" class="btn btn-secondary" @click.prevent="logout()"
                    >Logout</a
                  >
                </div>
                <div v-else>
                  <DatasetSummary />
                </div>
              </div>
            </div>

            <div class="card-body">
              <div>
                <div v-if="readmeLoading" class="text-primary">
                  <span class="sr-only">Loading...</span>
                </div>
                <div v-else>
                  <div v-if="readmeErrored">
                    <p>Unable to load readme please try again.</p>
                    <a
                      href=""
                      class="btn btn-secondary"
                      @click.prevent="updateReadme()"
                      >Try again</a
                    >
                    <p>Or try logging out and in again.</p>
                    <a
                      href=""
                      class="btn btn-secondary"
                      @click.prevent="logout()"
                      >Logout</a
                    >
                  </div>
                  <div v-else>
                    <Readme :getinfo="getinfo" />
                  </div>
                </div>

                <div v-if="annotationsLoading" class="text-primary">
                  <span class="sr-only">Loading...</span>
                </div>
                <div v-else>
                  <div v-if="annotationsErrored">
                    <p>Unable to load annotations please try again.</p>
                    <a
                      href=""
                      class="btn btn-secondary"
                      @click.prevent="updateAnnotations()"
                      >Try again</a
                    >
                    <p>Or try logging out and in again.</p>
                    <a
                      href=""
                      class="btn btn-secondary"
                      @click.prevent="logout()"
                      >Logout</a
                    >
                  </div>
                  <div v-else>
                    <Annotations />
                  </div>
                </div>
              </div>
            </div>

            <div class="card-footer">
              <div v-if="manifestLoading" class="text-primary">
                <span class="sr-only">Loading...</span>
              </div>
              <div v-else>
                <div v-if="manifestErrored">
                  <p>Unable to load manifest please try again.</p>
                  <a
                    href=""
                    class="btn btn-secondary"
                    @click.prevent="updateManifest()"
                    >Try again</a
                  >
                  <p>Or try logging out and in again.</p>
                  <a href="" class="btn btn-secondary" @click.prevent="logout()"
                    >Logout</a
                  >
                </div>
                <div v-else>
                  <Manifest />
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
    <div v-else>
      <SignIn @sign-in="setTokenAndSearch" />
    </div>
  </div>
</template>

<script>
import SignIn from "./components/SignIn.vue";
import SummaryInfo from "./components/SummaryInfo.vue";
import TextSearch from "./components/TextSearch.vue";
import DatasetTable from "./components/DatasetTable.vue";
import Manifest from "./components/DatasetManifest.vue";
import Readme from "./components/DatasetReadme.vue";
import Annotations from "./components/DatasetAnnotations.vue";
import DatasetSummary from "./components/DatasetSummary.vue";


export default {
  name: "app",
  data: function () {
    return {
      datasetHits: [],
      searchLoading: true,
      searchErrored: false,
      manifestLoading: false,
      manifestErrored: false,
      readmeLoading: false,
      readmeErrored: false,
      annotationsLoading: false,
      annotationsErrored: false,
      lookup_url: process.env.VUE_APP_DTOOL_LOOKUP_SERVER_URL,
      token: null,
      perPage: this.$store.state.update_current_Per_Page,
      pageNumber: 1,

      responseheaders: Array,
      getinfo: {},
    };
  },
  computed: {
    datasetLoaded: function () {
      return this.$store.state.current_dataset;
    },
    current_dataset: function () {
      return this.datasetHits[this.$store.state.current_dataset_index];
    },
    searchURL: function () {
      return (
        this.lookup_url +
        "/dataset/search?page=" +
        this.pageNumber +
        "&page_size=" +
        this.$store.state.update_current_Per_Page
      );
    },
    mongoSearchURL: function () {
      return this.lookup_url + "/mongo/query";
    },
    manifestURL: function () {
      return this.lookup_url + "/dataset/manifest";
    },
    configInfoURL: function () {
      return this.lookup_url + "/config/versions";
    },
    readmeURL: function () {
      return this.lookup_url + "/dataset/readme";
    },
    annotationsURL: function () {
      return this.lookup_url + "/dataset/annotations";
    },
    auth_str: function () {
      return "Bearer ".concat(this.token);
    },
    searchQuery: function () {
      var query = {};

      if (this.$store.state.mongo_text) {
        query.query = JSON.parse(this.$store.state.mongo_text);
      } else {
        if (this.$store.state.free_text) {
          query.free_text = this.$store.state.free_text;
        }

        if (this.$store.state.creator_usernames.length > 0) {
          query.creator_usernames = this.$store.state.creator_usernames;
        }
        if (this.$store.state.base_uris.length > 0) {
          query.base_uris = this.$store.state.base_uris;
        }
        if (this.$store.state.tags.length > 0) {
          query.tags = this.$store.state.tags;
        }
      }
      return query;
    },
    uriQuery: function () {
      if (this.datasetHits.length > 0) {
        return {
          uri: this.datasetHits[this.$store.state.current_dataset_index].uri,
        };
      } else {
        return { uri: null };
      }
    },
    pagination: function () {
      return this.responseheaders["x-pagination"]
        ? JSON.parse(this.responseheaders["x-pagination"])
        : {};
    },

    totalPageContents() {
      return this.pagination.total;
    },

    shouldShowPagination() {
      return (
        this.getinfo["dtool_lookup_server"] >= this.$store.state.current_required_version
      );
    },
  },
  methods: {
    setTokenAndSearch: function (token) {
      this.token = token;
      this.searchDatasets();
    },
    searchDatasets: function () {
      this.getconfiginfo(), console.log("Running search");
      console.log(this.searchQuery);
      this.$store.commit("update_current_dataset_index", 0);
      this.$store.commit("update_current_dataset", null);
      this.$store.commit("update_current_dataset_manifest", null);
      this.$store.commit("update_current_dataset_readme", null);
      this.updateDataset();
      this.searchLoading = true;
      this.searchErrored = false;

      let searchURL = this.searchURL;
      if (this.$store.state.mongo_text) {
        searchURL = this.mongoSearchURL;
      }

      this.$http
        .post(searchURL, this.searchQuery, {
          headers: {
            Authorization: this.auth_str,
            "Content-Type": "application/json",
          },
        })
        .then((response) => {
          this.datasetHits = response.data;
          this.responseheaders = response.headers;
          this.$store.commit("update_current_dataset", this.current_dataset);
          this.$store.commit("update_num_filtered", this.datasetHits.length);
          this.updateDataset();
        })
        .catch((error) => {
          console.log(error);
          console.log(error.response);
          this.searchErrored = true;
        })
        .finally(() => {
          this.searchLoading = false;
        });
    },
    updateDataset: function () {
      this.updateManifest();
      this.updateReadme();
      this.updateAnnotations();
    },
    updateManifest: function () {
      console.log("Loading manifest");
      console.log(this.uriQuery);
      this.manifestLoading = true;
      this.manifestErrored = false;
      this.$http
        .post(this.manifestURL, this.uriQuery, {
          headers: {
            Authorization: this.auth_str,
            "Content-Type": "application/json",
          },
        })
        .then((response) => {
          this.$store.commit("update_current_dataset_manifest", response.data);
        })
        .catch((error) => {
          console.log(error);
          console.log(error.response);
          this.manifestErrored = true;
        })
        .finally(() => (this.manifestLoading = false));
    },
    updateReadme: function () {
      console.log("Loading readme");
      console.log(this.uriQuery);
      this.readmeLoading = true;
      this.readmeErrored = false;
      this.$http
        .post(this.readmeURL, this.uriQuery, {
          headers: {
            Authorization: this.auth_str,
            "Content-Type": "application/json",
          },
        })
        .then((response) => {
          this.$store.commit("update_current_dataset_readme", response.data);
        })
        .catch((error) => {
          console.log(error);
          console.log(error.response);
          this.readmeErrored = true;
        })
        .finally(() => (this.readmeLoading = false));
    },
    updateAnnotations: function () {
      console.log("Loading annotations");
      console.log(this.uriQuery);
      this.annotationsLoading = true;
      this.annotationsErrored = false;
      this.$http
        .post(this.annotationsURL, this.uriQuery, {
          headers: {
            Authorization: this.auth_str,
            "Content-Type": "application/json",
          },
        })
        .then((response) => {
          this.$store.commit(
            "update_current_dataset_annotations",
            response.data
          );
        })
        .catch((error) => {
          console.log(error);
          console.log(error.response);
          this.annotationsErrored = true;
        })
        .finally(() => (this.annotationsLoading = false));
    },
    getconfiginfo: function () {
      console.log("Loading ConfigInfo");

      this.$http
        .get(this.configInfoURL, {
          headers: {
            Authorization: this.auth_str,
            "Content-Type": "application/json",
          },
        })
        .then((response) => {
          this.getinfo = response.data;
        })
        .catch((error) => {
          console.log(error);
          console.log(error.response);
        });
    },
    logout: function () {
      this.token = "";
      this.$store.commit("clear_all");
    },
  },

  components: {
    SignIn,
    SummaryInfo,
    TextSearch,
    DatasetTable,
    Manifest,
    Readme,
    Annotations,
    DatasetSummary,
    
  },
};
</script>

<style>
/* #app {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
} */

/*Set the row height to the viewport*/
.row-height {
  height: calc(100vh - 60px);
}

/*Set up the columns with a 100% height, body color and overflow scroll*/
</style>

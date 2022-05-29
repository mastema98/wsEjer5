<template>
  <b-container fluid>
    <!-- User Interface controls -->
    <b-row>
      <b-form-input
        v-model="filter"
        type="search"
        placeholder="Type to filter data"
      ></b-form-input>

      <!--Selector de cantidad de elementos por página
        <b-col sm="5" md="6" class="my-1">
        <b-form-group
          label="Elements per page"
          label-for="per-page-select"
          label-cols-sm="6"
          label-cols-md="4"
          label-cols-lg="3"
          label-align-sm="right"
          label-size="sm"
          class="mb-0"
        >
          <b-form-select
            id="per-page-select"
            v-model="perPage"
            :options="pageOptions"
            size="sm"
          ></b-form-select>
        </b-form-group>
      </b-col>-->
      <b-row>
        <b-col>
          <b-form-group
            label="Property"
            label-for="per-page-select"
            label-cols-sm="6"
            label-cols-md="4"
            label-cols-lg="3"
            label-align-sm="right"
            label-size="sm"
            class="mb-0"
          >
            <select v-model="firstOption">
              <option v-for="(item, index) in list" :key="index">
                {{ index }}
              </option>
            </select>
          </b-form-group>
        </b-col>
        <b-col>
          <b-form-group
            label="Value of property"
            label-for="per-page-select"
            label-cols-sm="6"
            label-cols-md="4"
            label-cols-lg="3"
            label-align-sm="right"
            label-size="sm"
            class="mb-0"
          >
            <select v-model="secondOption" v-if="firstOption">
              <option
                v-for="option in list[firstOption]"
                :value="option.ent"
                :key="option"
              >
                {{ option.name }}
              </option>
            </select>
          </b-form-group>
        </b-col>
        <b-col>
          <button v-on:click="search">Search</button>
        </b-col>
      </b-row>
    </b-row>

    <!-- Main table element -->
    <b-table
      ref="table"
      :items="items"
      :fields="fields"
      :current-page="currentPage"
      :per-page="perPage"
      :filter="filter"
      :filter-included-fields="filterOn"
      :sort-by.sync="sortBy"
      :sort-desc.sync="sortDesc"
      :sort-direction="sortDirection"
      stacked="md"
      show-empty
      small
      striped
      responsive
      @filtered="onFiltered"
    >
      <template #cell(name)="row">
        {{ row.value.first }} {{ row.value.last }}
      </template>

      <template #cell(actions)="row">
        <b-button size="sm" @click="row.toggleDetails">
          {{ row.detailsShowing ? "Hide" : "Show" }} Details
        </b-button>
      </template>

      <template #row-details="row">
        <b-card>
          <h1>{{ row.item.nem }}</h1>
          <p>Descripción: {{ row.item.description }}</p>
          <p>Escala: {{ row.item.scale }}</p>
          <p>Longitud: {{ row.item.length }} millas</p>
          <p>Anchura: {{ row.item.width }} yardas</p>
          <p>Fecha de inicio: {{ row.item.begDate }}</p>
          <p>Fecha de fin: {{ row.item.endDate }}</p>
          <p>Localización: {{ row.item.county }}</p>
          <p>
            Coordenadas iniciales: {{ row.item.begCoor.latitude }}/{{
              row.item.begCoor.longitude
            }}
          </p>
          <p>
            Coordenadas finales: {{ row.item.endCoor.latitude }}/{{
              row.item.endCoor.longitude
            }}
          </p>
          <div>
            <h3>Daños materiales:</h3>
            <p>Daños a la cosecha: {{ row.item.crop }} $</p>
            <p>Daños a la propiedad: {{ row.item.property }} $</p>
          </div>
          <div>
            <h3>Daños personales:</h3>
            <p>Muertes: {{ row.item.deaths }}</p>
            <p>Heridos: {{ row.item.injuries }}</p>
          </div>
          <p>Fuente: {{ row.item.source }}</p>
          <div>
            <h3>Eventos relacionados:</h3>
            <p>Tornados: {{ row.item.tornados }}</p>
            <p>Inundaciones repentinas: {{ row.item.flashFloods }}</p>
            <p>Inundaciones: {{ row.item.floods }}</p>
            <p>Vientos tormentosos: {{ row.item.thunderstormWinds }}</p>
            <p>Granizos: {{ row.item.hail }}</p>
            <p>Rayos: {{ row.item.lightnings }}</p>
            <p>Nubes embudo: {{ row.item.funnelClouds }}</p>
          </div>
          <div style="width: 100%">
            <img
              width="600"
              height="400"
              :src="row.item.src"
              :alt="row.item.nem"
            />
          </div>
        </b-card>
      </template>
    </b-table>

    <b-pagination
      v-model="currentPage"
      :total-rows="totalRows"
      :per-page="perPage"
    ></b-pagination>
  </b-container>
</template>

<script>
var tornados = [];
export default {
  data() {
    return {
      items: [],
      fields: [
        {
          key: "nem",
          label: "name",
          sortable: true,
          sortDirection: "desc",
        },
        {
          key: "scale",
          label: "scale",
          sortable: true,
          class: "text-center",
        },
        { key: "actions", label: "Actions" },
      ],
      totalRows: 1,
      currentPage: 1,
      perPage: 10,
      pageOptions: [
        5,
        10,
        25,
        50,
        100,
        { value: Number.MAX_SAFE_INTEGER, text: "show all" },
      ],
      sortBy: "",
      sortDesc: false,
      sortDirection: "asc",
      filter: null,
      filterOn: ["nem", "scale"],
      firstOption: null,
      secondOption: null,
      list: {
        Scale: [
          { name: "EF0", ent: "Q3" },
          { name: "EF1", ent: "Q4" },
          { name: "EF2", ent: "Q5" },
          { name: "EF3", ent: "Q6" },
          { name: "EF4", ent: "Q7" },
          { name: "EF5", ent: "Q8" },
        ],
        State: [
          { name: "Kansas", ent: "Q27" },
          { name: "Iowa", ent: "Q12" },
          { name: "Texas", ent: "Q38" },
          { name: "Oklahoma", ent: "Q50" },
          { name: "Georgia", ent: "Q32" },
        ],
        County: [
          { name: "Stanton county", ent: "Q43" },
          { name: "Butler county", ent: "Q13" },
          { name: "Linn county", ent: "Q28" },
          { name: "Dallas county", ent: "Q37" },
          { name: "Caddo county", ent: "Q51" },
          { name: "Bibb county", ent: "Q31" },
        ],
      },
    };
  },
  mounted() {
    const WBK = require("wikibase-sdk");
    const wbk = WBK({
      instance: "http://156.35.98.107:8080",
      sparqlEndpoint: "http://156.35.98.107:9999/bigdata/namespace/wdq/sparql",
    });
    const url = wbk.getReverseClaims("P9", [
      "Q3",
      "Q4",
      "Q5",
      "Q6",
      "Q7",
      "Q8",
    ]);
    this.search(url);
  },
  methods: {
    onFiltered(filteredItems) {
      // Trigger pagination to update the number of buttons/pages due to filtering
      this.totalRows = filteredItems.length;
      this.currentPage = 1;
    },
    getProp(entity) {
      const WBK = require("wikibase-sdk");
      const wbk = WBK({
        instance: "http://156.35.98.107:8080",
        sparqlEndpoint:
          "http://156.35.98.107:9999/bigdata/namespace/wdq/sparql",
      });
      var axios = require("axios");
      var config = {
        headers: {
          "Content-type": "application/json; charset=utf-8",
          "Access-Control-Allow-Origin": "*",
          "Access-Control-Allow-Methods": "*",
          "Access-Control-Allow-Headers": "*",
          "cache-control": "no-cache",
        },
      };
      var tornado = {};
      tornado["label"] = entity.labels.en.value;
      tornado["desc"] = entity.descriptions.en.value;
      //console.log(claim[0].mainsnak);
      Object.values(entity.claims).forEach((claim) => {
        if (claim[0].mainsnak.datavalue.type == "wikibase-entityid") {
          const url = wbk.getEntities(claim[0].mainsnak.datavalue.value.id);
          axios.get(url, config).then((response) => {
            const entities =
              response.data.entities[claim[0].mainsnak.datavalue.value.id];
            //console.log(entities);
            if (Object.keys(entities.claims).length == 0) {
              tornado[claim[0].mainsnak.property] = entities.labels.en;
            } else {
              Object.values(entities.claims).forEach((claim) => {
                if (claim[0].mainsnak.datavalue.type == "wikibase-entityid") {
                  const url = wbk.getEntities(
                    claim[0].mainsnak.datavalue.value.id
                  );
                  axios.get(url, config).then((response) => {
                    const entities =
                      response.data.entities[
                        claim[0].mainsnak.datavalue.value.id
                      ];
                    //console.log(entities);
                    if (Object.keys(entities.claims).length == 0) {
                      tornado[claim[0].mainsnak.property] = entities.labels.en;
                    } else {
                      Object.values(entities.claims).forEach((claim) => {
                        if (
                          claim[0].mainsnak.datavalue.type ==
                          "wikibase-entityid"
                        ) {
                          const url = wbk.getEntities(
                            claim[0].mainsnak.datavalue.value.id
                          );
                          axios.get(url, config).then((response) => {
                            const entities =
                              response.data.entities[
                                claim[0].mainsnak.datavalue.value.id
                              ];
                            //console.log(entities);
                            if (Object.keys(entities.claims).length == 0) {
                              tornado[claim[0].mainsnak.property] =
                                entities.labels.en;
                            } else {
                              Object.values(entities.claims).forEach(() => {});
                            }
                          });
                        } else {
                          tornado[claim[0].mainsnak.property] =
                            claim[0].mainsnak.datavalue;
                        }
                      });
                    }
                  });
                } else {
                  tornado[claim[0].mainsnak.property] =
                    claim[0].mainsnak.datavalue;
                }
              });
            }
          });
        } else {
          tornado[claim[0].mainsnak.property] = claim[0].mainsnak.datavalue;
        }
      });
      return tornado;
    },
    search(url = null) {
      if (
        (this.firstOption != null && this.secondOption != null) ||
        url != null
      ) {
        const WBK = require("wikibase-sdk");
        const wbk = WBK({
          instance: "http://156.35.98.107:8080",
          sparqlEndpoint:
            "http://156.35.98.107:9999/bigdata/namespace/wdq/sparql",
        });
        var axios = require("axios");
        var config = {
          headers: {
            "Content-type": "application/json; charset=utf-8",
            "Access-Control-Allow-Origin": "*",
            "Access-Control-Allow-Methods": "*",
            "Access-Control-Allow-Headers": "*",
            "cache-control": "no-cache",
          },
        };
        if (typeof url == "object") {
          const propPs = { Scale: "P9", State: "P12", County: "P13" };
          url = wbk.getReverseClaims(
            propPs[this.firstOption],
            this.secondOption
          );
        }
        tornados = [];
        axios.get(url, config).then((response) => {
          const entitiesIds = wbk.simplify.sparqlResults(response.data);
          var ids = [];
          JSON.parse(
            JSON.stringify(entitiesIds).replaceAll("subject", "entityId")
          ).forEach((entity) => {
            ids.push(entity.entityId);
          });
          if (this.firstOption == "State" || this.firstOption == "County") {
            const url3 = wbk.getReverseClaims("P5", ids);
            axios.get(url3, config).then((response) => {
              const entitiesIds = wbk.simplify.sparqlResults(response.data);
              var ids = [];
              JSON.parse(
                JSON.stringify(entitiesIds).replaceAll("subject", "entityId")
              ).forEach((entity) => {
                ids.push(entity.entityId);
              });
              const url2 = wbk.getEntities(ids);
              axios.get(url2, config).then((response2) => {
                ids.forEach((id) => {
                  tornados.push(this.getProp(response2.data.entities[id]));
                });
                console.log(tornados);
                this.delay(5000).then(() => this.pretty());
              });
            });
          } else {
            const url2 = wbk.getEntities(ids);
            axios.get(url2, config).then((response2) => {
              ids.forEach((id) => {
                tornados.push(this.getProp(response2.data.entities[id]));
              });
              console.log(tornados);
              this.delay(5000).then(() => this.pretty());
            });
          }
        });
      }
    },
    pretty() {
      var equiv = {
        P1: "width",
        P2: "length",
        P3: "begDate",
        P4: "endDate",
        P9: "scale",
        P10: "county",
        P11: "wikibase",
        P14: "deaths",
        P15: "injuries",
        P16: "tornados",
        P17: "flashFloods",
        P18: "floods",
        P19: "thunderstormWinds",
        P20: "hail",
        P21: "lightnings",
        P22: "funnelClouds",
        P23: "crop",
        P24: "property",
        P26: "source",
        P27: "begCoor",
        P28: "endCoor",
      };
      this.items.splice(0, this.items.length);
      tornados.forEach((tornado) => {
        const newRow = this.fields.reduce(
          (a, c) => ({ ...a, [c.key]: null }),
          {}
        );
        for (const [key, value] of Object.entries(equiv)) {
          newRow[value] = tornado[key].value;
          if (newRow[value].amount != undefined)
            newRow[value] = newRow[value].amount.replace("+", "");
        }
        newRow.nem = tornado["label"];
        newRow.description = tornado["desc"];
        newRow.src =
          "https://maps.geoapify.com/v1/staticmap?style=osm-carto&width=600&height=400&center=lonlat:" +
          newRow.begCoor.longitude +
          "," +
          newRow.begCoor.latitude +
          "&zoom=8&marker=lonlat:" +
          newRow.begCoor.longitude +
          "," +
          newRow.begCoor.latitude +
          ";color:%23ff0000;size:medium;text:Beg|lonlat:" +
          newRow.endCoor.longitude +
          "," +
          newRow.endCoor.latitude +
          ";color:%23ff0000;size:medium;text:End&apiKey=6cc19be3dddf4768ba8f0eaaf4de4dd6";

        this.items.unshift(newRow);
      });
    },
    delay(time) {
      return new Promise((resolve) => setTimeout(resolve, time));
    },
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.pagination {
  display: flex;
  justify-content: center;
}
</style>

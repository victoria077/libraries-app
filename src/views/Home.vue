<template>
  <div class="container">
    <div class="settings">
      <div class="settings__select">
        <select v-model="filter" @change="filterPages($event)">
          <option disabled value="">Фильтровать</option>
          <optgroup label="Выберите город">
            <option value="203">Санкт-Петербург</option>
            <option value="1643">Ростов-на-Дону</option>
            <option value="552">Воронеж</option>
            <option value="2248">Ставрополь</option>
          </optgroup>
        </select>
        <button @click="reset">X</button>
      </div>
      <div class="settings__select">
        <select v-model="sort" @change="sortPages($event)">
          <option disabled value="">Cортировать</option>
          <option value="ascending">От А до Я</option>
          <option value="descending">От Я до А</option>
        </select>
        <button @click="reset">X</button>
      </div>
    </div>
    <table class="table">
      <thead>
        <tr>
          <th>Название</th>
          <th>Адрес</th>
          <th>Часы работы</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="item in this.data" :key="item._id">
          <td data-label="Название">
            <p>{{ item.data.general.name }}</p>
            <router-link :to="'/library/' + item._id"
              >Узнать подробнее</router-link
            >
          </td>
          <td data-label="Адрес">
            {{ item.data.general.address.fullAddress }}
          </td>
          <td data-label="Часы работы">
            <div>
              <p v-if="item.data.general.workingSchedule[1]">
                пн-пт:
                {{ item.data.general.workingSchedule[1].from.substr(0, 5) }} -
                {{ item.data.general.workingSchedule[1].to.substr(0, 5) }}
              </p>
              <p v-if="item.data.general.workingSchedule[6]">
                сб:
                {{ item.data.general.workingSchedule[6].from.substr(0, 5) }}
                - {{ item.data.general.workingSchedule[6].to.substr(0, 5) }}
              </p>
            </div>
          </td>
        </tr>
      </tbody>
    </table>
    <div class="pagination">
      <button @click="prevPage" class="pagination__button">Назад</button>
      <p class="pagination__text">{{ this.page }}</p>
      <button @click="nextPage" class="pagination__button">Вперед</button>
    </div>
  </div>
</template>

<script>
// @ is an alias to /src
import axios from "axios";

export default {
  name: "Home",
  components: {},
  data() {
    return {
      data: [],
      confs: null,
      links: null,
      filter: "",
      sort: "",
      header: null,
      page: 1,
    };
  },
  async created() {
    try {
      fetch("/config.json")
        .then((res) => res.json())
        .then(async (config) => {
          this.confs = config;
          const res = await axios.get(`${this.confs.hostUrl}_page=1`);
          this.data = res.data;
          this.header = res.headers.link;
        });
    } catch (e) {
      console.error(e);
    }
  },
  computed: {
    linksPages() {
      let links = this.header;
      let arrData = links.split("link:");
      links = arrData.length == 2 ? arrData[1] : links;
      let parsed_data = {};
      arrData = links.split(",");
      for (let d of arrData) {
        let linkInfo = /<([^>]+)>;\s+rel="([^"]+)"/gi.exec(d);
        parsed_data[linkInfo[2]] = linkInfo[1];
      }
      return parsed_data;
    },
  },
  methods: {
    nextPage() {
      axios.get(this.linksPages.next).then((response) => {
        this.data = response.data;
        this.header = response.headers.link;
        this.page++;
      });
    },
    prevPage() {
      if (this.page > 1) {
        axios.get(this.linksPages.prev).then((response) => {
          this.data = response.data;
          this.header = response.headers.link;
          this.page--;
        });
      }
    },
    filterPages(event) {
      let url = `${this.confs.hostUrl}data.general.locale.id=${event.target.value}&_page=1`;
      axios
        .get(url, { headers: { "Content-Type": "text/plain; charset=utf-8" } })
        .then((response) => {
          this.data = response.data;
          this.header = response.headers.link;
          this.page = 1;
        });
    },
    sortPages(event) {
      if (event.target.value === "ascending") {
        axios
          .get(
            `${this.confs.hostUrl}_sort=data.general.locale.name&_order=asc&_page=1`
          )
          .then((resp) => {
            this.data = resp.data;
            this.header = resp.headers.link;
            this.page = 1;
          })
          .catch((error) => {
            console.log(error);
          });
      }else{
            axios.get(`${this.confs.hostUrl}_sort=data.general.locale.name&_order=desc&_page=1`)
                    .then(resp => {
                        this.data = resp.data;
                        this.header = resp.headers.link;
                        this.page = 1;
                    }).catch(error => {
                    console.log(error);
                });
      }
    },
    reset() {
      axios
        .get(`${this.confs.hostUrl}_page=1`)
        .then((resp) => {
          this.data = resp.data;
          this.header = resp.headers.link;
          this.page = 1;
        })
        .catch((error) => {
          console.log(error);
        });
    },
  },
};
</script>

<style scoped lang="scss">
.container {
  text-align: center;
  padding: 20px 70px;
  background-color: #f0f0f0;
  font-family: Apple;

  .table {
    width: 100%;
    background-color: white;
    border-collapse: collapse;
    border: 2px solid #e0e0e0;
    border-bottom: none;

    td,
    th {
      padding: 12px 15px;
      text-align: left;
      font-size: 20px;
    }

    th {
      color: #1d1d1f;
      border-bottom: 1px solid #ddd;
    }

    td {
      text-align: left;
    }

    tbody tr:nth-child(even) {
      background-color: #f0f0f0;
    }
  }

  .settings {
    display: flex;
    justify-content: flex-end;
    margin-bottom: 20px;

    &__sort {
      width: 130px;
      height: 40px;
      font-size: 18px;
      display: inline-block;
      outline: none;
      cursor: pointer;
      border-color: #dcdcdc !important;
      background-color: #979797;
      color: #fff;
      border-radius: 3px;
    }

    &__select {
      display: flex;
      flex-direction: row;

      select {
        width: 180px;
        height: 40px;
        font-size: 18px;
        -moz-border-radius-bottomleft: 3px;
        -moz-border-radius-topleft: 3px;
        display: inline-block;
        outline: none;
        cursor: pointer;
        background-color: #fff;
        border: none;
        color: #979797;
        padding-left: 10px;
      }

      button {
        margin-right: 20px;
        height: 40px;
        width: 40px;
        background-color: #fff;
        color: #979797;
        border: none;
      }
    }
  }

  .pagination {
    display: flex;
    flex-direction: row;
    justify-content: center;
    align-items: center;

    &__button {
      width: 100px;
      height: 40px;
      font-size: 18px;
      outline: none;
      cursor: pointer;
      margin: 25px 0px 30px 0;
      border: none;
      background-color: #979797;
      color: #fff;
      border-radius: 3px;

      &:hover {
        background-color: #e0e0e0;
        color: #979797;
      }
    }

    &__text {
      font-size: 20px;
      margin: 0 20px;
      color: #979797;
    }
  }
}

@media (max-width: 500px) {
  .container {
    padding: 20px;
    font-size: 16px;
    word-wrap: break-word;

    .table {
      display: block;
      width: 100%;

      tbody,
      tr,
      td {
        display: block;
        width: 100%;
      }

      thead {
        display: none;
      }

      tr {
        margin-bottom: 15px;
      }

      td {
        text-align: right;
        padding-left: 50%;
        text-align: right;
        position: relative;
        font-size: 16px;
      }

      td::before {
        content: attr(data-label);
        position: absolute;
        left: 0;
        width: 50%;
        padding-left: 15px;
        font-size: 15px;
        font-weight: bold;
        text-align: left;
      }
    }

    .settings {
      &__select {
        select {
          width: 100px;
          font-size: 14px;
        }
      }

      &__sort {
        font-size: 14px;
      }
    }
  }
}

@media (max-width: 330px) {
  .container {
    .settings {
      &__select {
        select {
          max-width: 90px;
        }
      }
    }
  }
}
</style>

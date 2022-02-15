<template>
    <div>
        <div class="panel panel-default">
            <div class="h1 m-3">
                Articles list
                <button
                    :to="{name: 'mewParsing'}"
                    v-on:click="addSites($event)"
                    class="btn btn-xs btn-outline-primary float-end"
                >
                    New parsing
                </button>
            </div>

            <div class="panel-body">
                <table class="table table-bordered table-striped">
                    <thead>
                    <tr>
                        <th>ID</th>
                        <th>Title</th>
                        <th>Points</th>
                        <th>Item ID</th>
                        <th>Created</th>
                        <th width="150">&nbsp;</th>
                    </tr>
                    </thead>
                    <tbody>
                    <tr v-for="item, index in items" v-bind:key="item.id" class="align-baseline">
                        <td class="align-content-center">{{ item.id }}</td>
                        <td>
                            <a v-bind:href="item.link">
                                {{ item.title }}
                            </a>

                        </td>
                        <td class="align-content-center">
                            <span :id="'point_' + item.id_item">{{ item.points }}</span>
                        </td>
                        <td>{{ item.id_item }}</td>
                        <td>{{ item.created }}</td>
                        <td>
                            <button
                               class="btn btn-xs btn-outline-primary"
                               v-on:click="updatePoints(item.id_item, index, $event)">
                                Update points
                            </button>
                        </td>
                    </tr>
                    </tbody>
                </table>
                <pagination :data="laravelData" align="center" :showDisabled="true" :limit="3"
                            @pagination-change-page="getResults"></pagination>
            </div>
        </div>
    </div>
</template>

<script>
export default {
    name: "ParserIndex",
    data: function () {
        return {
            items: {},
            laravelData: {},
        }
    },
    created() {
        this.getResults();
    },
    methods: {
        getResults(page) {
            //var app = this;
            if (typeof page === "undefined") {
                page = 1;
            }
            axios.get("/api/v1/sites?page=" + page)
                .then(resp => {
                    this.items = resp.data.data;
                    this.laravelData = resp.data;
                });
        },
        updatePoints(id_item, index, e) {
            if (confirm("Do you really want to update it?")) {
                e.originalTarget.disabled = true;
                axios.get('/api/v1/parser/update-point/' + id_item)
                    .then(resp => {
                        if (resp.data.id_item !== false) {
                            this.items[index].points = resp.data.points;
                            alert("item " + resp.data.id_item + "  updated. New points are - " + resp.data.points);
                        } else {
                            alert("item " + resp.data.id_item + " is no longer available, so its points cannot be updated.");
                        }
                        e.originalTarget.disabled = false;
                    })
                    .catch(function (resp) {
                        alert("Could not update item");
                        e.originalTarget.disabled = false;
                    });
            }
        },
        addSites(e) {
            if (confirm("Do you really want to add sites?")) {
                e.originalTarget.disabled = true;
                var innerText = e.originalTarget.innerText;
                e.originalTarget.innerText = 'Parsing is on';
                axios.get('/api/v1/parser/add/')
                    .then(function (resp) {
                        location.reload();
                    })
                    .catch(function (resp) {
                        e.originalTarget.disabled = false;
                        e.originalTarget.innerText = innerText;
                        //alert("Could not add items");
                    });
            }
        }
    }
}
</script>

<style scoped>

</style>

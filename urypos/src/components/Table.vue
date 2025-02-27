<template>
  <div class="switch-wrapper mt-2">
    <input
      id="Table"
      type="radio"
      name="switch"
      v-model="this.table.selectedOption"
      value="Table"
    />
    <input
      id="take_away"
      type="radio"
      name="switch"
      v-model="this.table.selectedOption"
      value="take_away"
    />
    <label for="Table" class="switch-label text-center">Table</label>
    <label for="take_away" class="switch-label text-center">Takeaway</label>
    <span class="highlighter"></span>
  </div>
  <div v-if="this.table.selectedOption === 'Table'" class="m-auto">
    <div class="flow-root">
      <div
        class="fixed inset-0 z-50 flex items-center justify-center bg-gray-300 bg-opacity-50 text-lg"
        v-if="this.invoiceData.isPrinting"
      >
        Printing Invoice
      </div>
      <div class="grid grid-cols-2 gap-4 md:grid-cols-4 lg:grid-cols-5">
        <div
          w-full
          class="w-full max-w-sm rounded border border-gray-200 bg-white shadow dark:border-gray-700 dark:bg-gray-800"
          v-for="table in this.table.filteredTables"
          :key="table.name"
        >
          <div class="flex justify-between">
            <div class="flex justify-start px-2 pt-2">
              <span
                class="me-2 rounded px-2.5 py-0.5 text-sm font-medium"
                :class="{
                  'bg-red-100 text-red-800 dark:bg-red-900 dark:text-red-300':
                    this.table.getBadgeType(table) === 'red',
                  'bg-blue-100 text-blue-800 dark:bg-blue-900 dark:text-blue-300':
                    this.table.getBadgeType(table) === 'default',

                  'bg-yellow-100 text-yellow-800':
                    this.table.getBadgeType(table) === 'yellow',
                  'bg-green-100 text-green-800':
                    this.table.getBadgeType(table) === 'green',
                }"
              >
                {{ this.table.getBadgeText(table) }}
              </span>
            </div>
            <div class="relative" v-if="table.occupied === 1">
              <button
                class="inline-block rounded-lg p-1.5 text-sm text-gray-500 dark:text-gray-400 dark:hover:bg-gray-700 dark:focus:ring-gray-700"
                type="button"
                @click="this.table.toggleDropdown(table.name)"
              >
                <svg
                  class="h-6 w-6"
                  aria-hidden="true"
                  fill="currentColor"
                  viewBox="0 0 20 20"
                  xmlns="http://www.w3.org/2000/svg"
                >
                  <path
                    d="M6 10a2 2 0 11-4 0 2 2 0 014 0zM12 10a2 2 0 11-4 0 2 2 0 014 0zM16 12a2 2 0 100-4 2 2 0 000 4z"
                  ></path>
                </svg>
              </button>
              <div
                class="absolute right-0 z-10 w-36 divide-y divide-gray-100 rounded-lg bg-white shadow dark:bg-gray-700"
                v-show="this.table.activeDropdown === table.name"
              >
                <ul class="py-2">
                  <li>
                    <a
                      href="#"
                      class="block px-4 py-2 text-sm text-gray-700 hover:bg-gray-100 dark:text-gray-200 dark:hover:bg-gray-600 dark:hover:text-white"
                      @click="this.table.showModal = true"
                      >Table Transfer</a
                    >
                  </li>
                  <li v-if="this.auth.hasAccess">
                    <a
                      href="#"
                      class="block px-4 py-2 text-sm text-gray-700 hover:bg-gray-100 dark:text-gray-200 dark:hover:bg-gray-600 dark:hover:text-white"
                      @click="this.table.showModalCaptainTransfer = true"
                      >Captain Transfer</a
                    >
                  </li>
                </ul>
              </div>
            </div>
          </div>
          <div class="flex flex-col pb-4">
            <div
              class="mt-1 text-center"
              @click="
                table.occupied === 1 && !this.auth.restrictTableOrder
                  ? this.table.routeToMenu(table)
                  : ''
              "
            >
              <h5
                class="mt-2 text-xl font-medium text-gray-900 dark:text-white"
                :class="{ 'mt-3': table.occupied === 0 }"
              >
                {{ table.name }}
              </h5>
              <span class="text-sm text-gray-500 dark:text-gray-400">
                {{
                  table.occupied === 1
                    ? this.table.getTimeDifference(table)
                    : ""
                }}</span
              >
            </div>
            <div class="mt-8 text-center" v-if="table.occupied != 1">
              <button
                type="button"
                class="inline-flex items-center rounded px-2 py-2.5 text-center text-sm font-medium text-white hover:bg-[#2557D6]/90 focus:outline-none focus:ring-4 focus:ring-[#2557D6]/50 dark:focus:ring-[#2557D6]/50"
                :class="[
                  {
                    'bg-blue-700': !this.auth.restrictTableOrder,
                    'pointer-events-none bg-blue-400':
                      this.auth.restrictTableOrder,
                  },
                ]"
                @click="
                  !this.auth.restrictTableOrder &&
                    this.table.addToSelectedTables(table)
                "
              >
                Open Table
                <svg
                  class="ml-2 h-6 w-6 dark:text-white"
                  fill="none"
                  stroke="currentColor"
                  viewBox="0 0 24 24"
                  xmlns="http://www.w3.org/2000/svg"
                >
                  <path
                    stroke-linecap="round"
                    stroke-linejoin="round"
                    stroke-width="2"
                    d="M13 9l3 3m0 0l-3 3m3-3H8m13 0a9 9 0 11-18 0 9 9 0 0118 0z"
                  ></path>
                </svg>
              </button>
            </div>
            <div class="mt-2 flex justify-center" v-if="table.occupied === 1">
              <button
                type="button"
                class="mb-2 me-2 inline-flex items-center rounded bg-blue-700 px-5 py-2.5 text-center text-sm font-medium text-white hover:bg-[#2557D6]/90 focus:outline-none focus:ring-4 focus:ring-[#2557D6]/50 dark:focus:ring-[#2557D6]/50"
                @click="this.invoiceData.billing(table)"
              >
                <svg
                  class="svg-icon mr-2"
                  viewBox="0 0 24 24"
                  width="18"
                  height="18"
                  fill="white"
                >
                  <path
                    d="M6 19H3a1 1 0 0 1-1-1V8a1 1 0 0 1 1-1h3V3a1 1 0 0 1 1-1h10a1 1 0 0 1 1 1v4h3a1 1 0 0 1 1 1v10a1 1 0 0 1-1 1h-3v2a1 1 0 0 1-1 1H7a1 1 0 0 1-1-1v-2zm0-2v-1a1 1 0 0 1 1-1h10a1 1 0 0 1 1 1v1h2V9H4v8h2zM8 4v3h8V4H8zm0 13v3h8v-3H8zm-3-7h3v2H5v-2z"
                  />
                </svg>

                Bill
              </button>

              <div
                class="relative inline-flex h-10 w-10 items-center justify-center overflow-hidden rounded-full border hover:bg-blue-700 hover:text-white focus:outline-none focus:ring-4 focus:ring-blue-300 dark:border-blue-500 dark:text-blue-500 dark:hover:bg-blue-500 dark:hover:text-white dark:focus:ring-blue-800"
                :class="[
                  {
                    'border-blue-700 text-blue-700':
                      !this.auth.restrictTableOrder,
                    'pointer-events-none border-blue-400 text-blue-400':
                      this.auth.restrictTableOrder,
                  },
                ]"
                @click="
                  !this.auth.restrictTableOrder && this.table.routeToCart(table)
                "
              >
                <svg
                  aria-hidden="true"
                  class="h-10 w-6"
                  fill="currentColor"
                  viewBox="0 0 20 20"
                  xmlns="http://www.w3.org/2000/svg"
                >
                  <path d="M10 12a2 2 0 100-4 2 2 0 000 4z"></path>
                  <path
                    fill-rule="evenodd"
                    d="M.458 10C1.732 5.943 5.522 3 10 3s8.268 2.943 9.542 7c-1.274 4.057-5.064 7-9.542 7S1.732 14.057.458 10zM14 10a4 4 0 11-8 0 4 4 0 018 0z"
                    clip-rule="evenodd"
                  ></path>
                </svg>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
  <div
    v-if="this.table.selectedOption === 'take_away'"
    class="container m-auto"
  >
    <takeAwayTable />
  </div>

  <div
    v-if="table.showModal"
    class="fixed inset-0 z-10 overflow-y-auto bg-gray-100"
  >
    <div class="mt-20 flex items-center justify-center">
      <div class="mt-10 w-full rounded-lg bg-white p-6 shadow-lg md:max-w-md">
        <div class="flex justify-end">
          <span class="sr-only">Close</span>
          <svg
            xmlns="http://www.w3.org/2000/svg"
            class="h-5 w-5"
            fill="none"
            viewBox="0 0 24 24"
            stroke="currentColor"
            @click="this.table.showModal = false"
          >
            <path
              stroke-linecap="round"
              stroke-linejoin="round"
              stroke-width="2"
              d="M6 18L18 6M6 6l12 12"
            />
          </svg>
        </div>

        <h2
          class="mt-1 block text-left text-xl font-medium text-gray-900 dark:text-white"
        >
          Table Transfer
        </h2>
        <div class="relative" ref="container">
          <label
            for="newTable"
            class="mt-6 block text-left text-gray-900 dark:text-white"
          >
            New Table
          </label>
          <input
            type="text"
            class="mt-4 w-full appearance-none rounded border p-2 leading-tight text-gray-900 shadow focus:outline-none"
            v-model="table.newTable"
            @click="
              this.table.showTable = true;
              this.table.tableSearch();
            "
          />
          <div
            v-if="this.table.showTable"
            class="absolute left-0 top-full z-10 max-h-64 w-full overflow-y-scroll rounded bg-white shadow"
            ref="dropdown"
          >
            <div
              class="h-16 w-full rounded-lg p-4 hover:bg-gray-100"
              v-for="(tables, index) in this.table.searchTable"
              :key="index"
              @click="this.table.selectTable(tables)"
            >
              <h1 class="text-base font-semibold leading-normal">
                {{ tables.name }}
              </h1>
            </div>
          </div>
        </div>
        <label
          for="newTable"
          class="mt-6 block text-left text-gray-900 dark:text-white"
        >
          Current Table
        </label>
        <input
          type="text"
          id="newTable"
          class="mt-4 w-full appearance-none rounded border p-2 leading-tight text-gray-900 shadow focus:outline-none"
          :value="table.tableName"
          readonly
        />
        <div class="flex justify-end">
          <button
            @click="
              this.table.showModal = false;
              this.table.tableTransfer(table);
            "
            class="mt-8 rounded bg-blue-500 px-3 py-2 text-white hover:bg-blue-600"
          >
            Transfer
          </button>
        </div>
      </div>
    </div>
  </div>
  <div
    v-if="table.showModalCaptainTransfer"
    class="fixed inset-0 z-10 overflow-y-auto bg-gray-100"
  >
    <div class="mt-20 flex items-center justify-center">
      <div class="mt-10 w-full rounded-lg bg-white p-6 shadow-lg md:max-w-md">
        <div class="flex justify-end">
          <span class="sr-only">Close</span>
          <svg
            xmlns="http://www.w3.org/2000/svg"
            class="h-5 w-5"
            fill="none"
            viewBox="0 0 24 24"
            stroke="currentColor"
            @click="table.showModalCaptainTransfer = false"
          >
            <path
              stroke-linecap="round"
              stroke-linejoin="round"
              stroke-width="2"
              d="M6 18L18 6M6 6l12 12"
            />
          </svg>
        </div>
        <h2
          class="mt-1 block text-left text-xl font-medium text-gray-900 dark:text-white"
        >
          Captain Transfer
        </h2>
        <div class="relative" ref="container">
          <label
            for="newTable"
            class="mt-6 block text-left text-gray-900 dark:text-white"
          >
            New Captain
          </label>
          <input
            type="text"
            class="mt-4 w-full appearance-none rounded border p-2 leading-tight text-gray-900 shadow focus:outline-none"
            @click="
              this.table.showCaptain = true;
              this.table.fetchCaptain();
            "
            v-model="this.table.newCaptain"
          />
          <div
            v-if="this.table.showCaptain"
            class="absolute left-0 top-full z-10 max-h-64 w-full overflow-y-scroll rounded bg-white shadow"
            ref="dropdown"
          >
            <div
              class="h-16 w-full rounded-lg p-4 hover:bg-gray-100"
              v-for="(captain, index) in this.table.searchCaptian"
              :key="index"
              @click="this.table.selectcaptain(captain)"
            >
              <h1 class="text-base font-semibold leading-normal">
                {{ captain.name }}
              </h1>
            </div>
          </div>
        </div>
        <label
          for="newTable"
          class="mt-6 block text-left text-gray-900 dark:text-white"
        >
          Current Captain
        </label>
        <input
          type="text"
          id="newTable"
          class="mt-4 w-full appearance-none rounded border p-2 leading-tight text-gray-900 shadow focus:outline-none"
          :value="this.table.currentCaptain"
          readonly
        />
        <div class="flex justify-end">
          <button
            @click="
              this.table.showModalCaptainTransfer = false;
              this.table.captianTransfer();
            "
            class="mt-8 rounded bg-blue-500 px-3 py-2 text-white hover:bg-blue-600"
          >
            Transfer
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { useTableStore } from "@/stores/Table.js";
import { useInvoiceDataStore } from "@/stores/invoiceData.js";
import { useAuthStore } from "@/stores/Auth.js";
import takeAwayTable from "./takeAwayTable.vue";
export default {
  name: "Table",
  components: {
    takeAwayTable,
  },
  setup() {
    const table = useTableStore();
    const invoiceData = useInvoiceDataStore();
    const auth = useAuthStore();
    return { table, invoiceData, auth };
  },
};
</script>
<style scoped>
.switch-wrapper {
  display: inline-flex;
  position: relative;
  padding: 4px;
  border: 1px solid lightgray;
  margin-bottom: 15px;
  border-radius: 0.5rem;
  background: white;
}

.switch-wrapper [type="radio"] {
  position: absolute;
  left: -9999px;
}

.switch-wrapper [type="radio"]:checked#Table ~ label[for="Table"],
.switch-wrapper [type="radio"]:checked#take_away ~ label[for="take_away"] {
  color: white;
}

.switch-wrapper [type="radio"]:checked#Table ~ label[for="Table"]:hover,
.switch-wrapper
  [type="radio"]:checked#take_away
  ~ label[for="take_away"]:hover {
  background: transparent;
}

.switch-wrapper
  [type="radio"]:checked#Table
  + label[for="take_away"]
  ~ .highlighter {
  transform: none;
}

.switch-wrapper
  [type="radio"]:checked#take_away
  + label[for="Table"]
  ~ .highlighter {
  transform: translateX(100%);
}

.switch-wrapper label {
  font-size: 16px;
  z-index: 1;
  min-width: 100px;
  line-height: 32px;
  cursor: pointer;
  border-radius: 30px;
  transition: color 0.25s ease-in-out;
}

.switch-wrapper label:hover {
  background: whitesmoke;
}

.switch-wrapper .highlighter {
  position: absolute;
  top: 4px;
  left: 4px;
  width: calc(50% - 4px);
  height: calc(100% - 8px);
  border-radius: 0.2rem;
  background: #1b5ed9;
  transition: transform 0.25s ease-in-out;
}
</style>

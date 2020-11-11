<template>
  <div class="h-screen flex overflow-hidden bg-gray-100">
    <Sidenav />
    <div class="flex flex-col w-0 flex-1 overflow-hidden border border-r-0 border-t-0 border-b-0">
      <main
        class="flex-1 relative z-0 overflow-y-auto focus:outline-none mt-12 lg:mt-0"
        tabindex="0"
      >
        <div class="pt-2 pb-6 md:py-6">
          <div class="flex flex-col col-gap-4" :class="showPickupForm ? 'hidden' : 'block'">
            <div class="w-full mt-5 p-8">
              <div class="w-full border border-t-0 border-l-0 border-r-0">
                <span
                  class="w-full flex flex-col items-center text-gray-800 font-semibold text-2xl mb-3"
                >Available pickups</span>
              </div>

              <client-only>
                <div class="overflow-x-auto inline-block w-full shadow rounded-lg">
                  <Paginator
                    :maxVisibleButtons="3"
                    :currentPage="current_page"
                    :totalPages="Math.ceil(pickupData.length/limit)"
                    :total="pickupData.length"
                    @pageChanged="refetchPickups"
                  />
                  <table class="min-w-full leading-normal">
                    <thead>
                      <tr>
                        <th
                          class="px-5 py-3 border-b-2 border-gray-200 bg-gray-100 text-left text-xs font-semibold text-gray-600 uppercase tracking-wider"
                        >
                          S/N
                        </th>
                        <th
                          class="px-5 py-3 border-b-2 border-gray-200 bg-gray-100 text-left text-xs font-semibold text-gray-600 uppercase tracking-wider"
                        >
                          Name
                        </th>
                        <th
                          class="px-5 py-3 border-b-2 border-gray-200 bg-gray-100 text-left text-xs font-semibold text-gray-600 uppercase tracking-wider"
                        >
                          Title
                        </th>
                        <th
                          class="px-5 py-3 border-b-2 border-gray-200 bg-gray-100 text-left text-xs font-semibold text-gray-600 uppercase tracking-wider"
                        >
                          Description
                        </th>
                        <th
                          class="px-5 py-3 border-b-2 border-gray-200 bg-gray-100 text-left text-xs font-semibold text-gray-600 uppercase tracking-wider"
                        >
                          Active
                        </th>
                        <th
                          class="px-5 py-3 border-b-2 border-gray-200 bg-gray-100 text-left text-xs font-semibold text-gray-600 uppercase tracking-wider"
                        />
                        <th
                          class="px-5 py-3 border-b-2 border-gray-200 bg-gray-100 text-left text-xs font-semibold text-gray-600 uppercase tracking-wider"
                        />
                      </tr>
                    </thead>
                    <tbody>
                      <tr v-for="(pickup, index) in pickups" :key="index">
                        <td
                          class="px-4 py-2 border-b border-gray-200 bg-white text-sm"
                          :class="index%2 === 0 ? '' : 'bg-gray-100'"
                        >
                          {{ current_page === 1 ? index+1 : (limit*(current_page-1))+index+1 }}.
                        </td>
                        <td
                          class="px-1 py-1 font-ubuntu border-b border-gray-200 bg-white text-sm"
                          :class="index%2 === 0 ? '' : 'bg-gray-100'"
                        >
                          {{ pickup.name }}
                        </td>
                        <td
                          class="px-1 py-1 border-b border-gray-200 bg-white text-sm"
                          :class="index%2 === 0 ? '' : 'bg-gray-100'"
                        >
                          {{ pickup.title }}
                        </td>
                        <td
                          class="px-1 py-1 border-b border-gray-200 bg-white text-sm"
                          :class="index%2 === 0 ? '' : 'bg-gray-100'"
                        >
                          {{ pickup.description }}
                        </td>
                        <td
                          class="px-4 py-2 border-b border-gray-200 bg-white text-sm"
                          :class="index%2 === 0 ? '' : 'bg-gray-100'"
                        >
                          <button v-if="pickup.is_active" disabled class="rounded-full cursor-text text-green-500 py-1 px-2 border border-green-500 bg-transparent text-sm leading-5 font-medium">
                            active
                          </button>
                          <button v-else disabled class="rounded-full cursor-text text-gray-600 py-1 px-2 border border-gray-400 bg-transparent bg-gray-200 text-sm leading-5 font-medium">
                            inactive
                          </button>
                        </td>
                        <td class="py-1 px-1 border-b border-gray-200 bg-white text-sm">
                          <div class="flex flex-row">
                            <button
                              class="py-2 px-4 mr-1 border border-transparent text-sm leading-5 font-medium rounded text-green-500 bg-green-100 hover:border-green-500 focus:outline-none focus:shadow-outline-blue active:bg-gray-800 transition duration-150 ease-in-out"
                              @click="openModal(pickup.pickup_id)"
                            >
                              <font-awesome-icon :icon="['fa', 'edit']" />
                            </button>
                            <button
                              class="py-2 px-4 border border-transparent text-sm leading-5 font-medium rounded text-red-500 bg-red-100 hover:border-red-500 focus:outline-none focus:shadow-outline-blue active:bg-gray-800 transition duration-150 ease-in-out"
                              @click="deletePickup(pickup.pickup_id)"
                            >
                              <font-awesome-icon :icon="['fa', 'trash']" />
                            </button>
                          </div>
                        </td>
                        <td class="py-1 px-1 border-b border-gray-200 bg-white text-sm">
                          <div class="flex flex-row">
                            <button
                              class="py-2 px-4 border border-transparent text-sm leading-5 font-medium rounded text-gray-700 focus:outline-none focus:shadow-outline-blue active:bg-gray-800 transition duration-150 ease-in-out"
                              @click="toggleDropDown(pickup.pickup_id)"
                            >
                              <font-awesome-icon :icon="['fa', 'ellipsis-v']" />
                            </button>
                          </div>
                          <div :class="dropDownStateId == pickup.pickup_id ? 'block' : 'hidden'">
                            <button
                              type="button"
                              class="hidden sm:block sm:fixed sm:opacity-0 sm:inset-0 sm:w-full sm:h-full sm:cursor-default"
                              @click="toggleDropDown(pickup.pickup_id)"
                            />
                            <div
                              class="z-50 sm:mt-2 sm:absolute sm:bg-white sm:rounded sm:right-0 sm:shadow-xl sm:w-48 sm:py-2"
                            >
                              <span
                                class="py-3 block cursor-pointer text-gray-400 hover:text-white sm:text-sm sm:text-gray-800 sm:px-4 sm:py-2 sm:hover:bg-red-700"
                                @click="moveItemTo('top', pickup.pickup_id)"
                              >
                                Move to top
                              </span>
                              <span
                                class="py-3 block cursor-pointer text-gray-400 hover:text-white sm:text-sm sm:text-gray-800 sm:px-4 sm:py-2 sm:hover:bg-red-700"
                                @click="moveItemTo('bottom', pickup.pickup_id)"
                              >
                                Move to bottom
                              </span>
                            </div>
                          </div>
                        </td>
                      </tr>
                    </tbody>
                  </table>

                  <!-- Modal -->
                  <b-modal
                    :active.sync="makeModal"
                    has-modal-card
                    trap-focus
                    :destroy-on-hide="false"
                    aria-role="dialog"
                    aria-modal
                  >
                    <div>
                      <div
                        class="flex items-center justify-center py-4 px-4 sm:px-6 lg:px-8 modal-card"
                      >
                        <div class="max-w-md w-full modal-card-body bg-gray-100">
                          <div>
                            <h2
                              class="mt-3 text-center text-3xl leading-9 font-extrabold text-gray-900"
                            >
                              Edit Pickup ({{ pickupObj.name }})
                            </h2>
                          </div>
                          <form class="mt-8" action="#" method="POST">
                            <input type="hidden" name="remember" value="true">
                            <div class="rounded-md shadow-sm">
                              <div class="grid grid-cols-6 w-full">
                                <div class="col-span-6 w-full">
                                  <b-tooltip
                                    label="Name of pickup location (internal use)"
                                    class="w-full"
                                    type="is-info"
                                    position="is-bottom"
                                  >
                                    <input
                                      id="pickup-name"
                                      v-model="editForm.name"
                                      placeholder="Pickup name"
                                      class="mt-2 form-input block w-full py-2 px-3 border border-gray-400 rounded-md shadow-sm focus:outline-none focus:shadow-outline-blue focus:border-blue-300 transition duration-150 ease-in-out sm:text-sm sm:leading-5"
                                    >
                                  </b-tooltip>
                                </div>
                                <div class="col-span-6 w-full">
                                  <b-tooltip
                                    label="The name displayed to your customers"
                                    class="w-full"
                                    type="is-info"
                                    position="is-bottom"
                                  >
                                    <input
                                      id="pickup-title"
                                      v-model="editForm.title"
                                      placeholder="Pickup title"
                                      class="mt-2 form-input block w-full py-2 px-3 border border-gray-400 rounded-md shadow-sm focus:outline-none focus:shadow-outline-blue focus:border-blue-300 transition duration-150 ease-in-out sm:text-sm sm:leading-5"
                                    >
                                  </b-tooltip>
                                </div>
                                <div class="col-span-6 w-full">
                                  <b-tooltip
                                    label="Describe the location"
                                    class="w-full"
                                    type="is-info"
                                    position="is-bottom"
                                  >
                                    <textarea
                                      id="pickup-description"
                                      v-model="editForm.description"
                                      placeholder="Pickup description"
                                      class="mt-2 form-input block w-full py-2 px-3 border border-gray-400 rounded-md shadow-sm focus:outline-none focus:shadow-outline-blue focus:border-blue-300 transition duration-150 ease-in-out sm:text-sm sm:leading-5"
                                    />
                                  </b-tooltip>
                                </div>
                                <div class="col-span-6 sm:col-span-3">
                                  <b-tooltip
                                    label="Zip/postcode of pickup location"
                                    class="w-full"
                                    type="is-info"
                                    position="is-bottom"
                                  >
                                    <input
                                      id="zip-code"
                                      v-model="editForm.location.zip"
                                      placeholder="Zip code"
                                      class="mt-2 form-input block w-full py-2 px-3 border border-gray-400 rounded-md shadow-sm focus:outline-none focus:shadow-outline-blue focus:border-blue-300 transition duration-150 ease-in-out sm:text-sm sm:leading-5"
                                    >
                                  </b-tooltip>
                                </div>
                                <div class="col-span-6 sm:col-span-3">
                                  <b-tooltip
                                    label="City where pickup location is sited"
                                    class="w-full"
                                    type="is-info"
                                    position="is-bottom"
                                  >
                                    <input
                                      id="city"
                                      v-model="editForm.location.city"
                                      placeholder="City"
                                      class="mt-2 ml-1 form-input block w-full py-2 px-3 border border-gray-400 rounded-md shadow-sm focus:outline-none focus:shadow-outline-blue focus:border-blue-300 transition duration-150 ease-in-out sm:text-sm sm:leading-5"
                                    >
                                  </b-tooltip>
                                </div>
                                <div class="col-span-6 w-full">
                                  <b-tooltip
                                    label="State where pickup location is sited"
                                    class="w-full"
                                    type="is-info"
                                    position="is-bottom"
                                  >
                                    <input
                                      id="state"
                                      v-model="editForm.location.state"
                                      placeholder="State"
                                      class="mt-2 form-input block w-full py-2 px-3 border border-gray-400 rounded-md shadow-sm focus:outline-none focus:shadow-outline-blue focus:border-blue-300 transition duration-150 ease-in-out sm:text-sm sm:leading-5"
                                    >
                                  </b-tooltip>
                                </div>
                                <div class="col-span-6 w-full">
                                  <b-tooltip
                                    label="Country where pickup location is sited"
                                    class="w-full"
                                    type="is-info"
                                    position="is-left"
                                  >
                                    <input
                                      id="Country"
                                      v-model="editForm.location.country"
                                      placeholder="Country"
                                      class="mt-2 form-input block w-full py-2 px-3 border border-gray-400 rounded-md shadow-sm focus:outline-none focus:shadow-outline-blue focus:border-blue-300 transition duration-150 ease-in-out sm:text-sm sm:leading-5"
                                    >
                                  </b-tooltip>
                                </div>
                                <div class="col-span-6 w-full">
                                  <b-tooltip
                                    label="Full address of the pickup location"
                                    class="w-full"
                                    type="is-info"
                                    position="is-bottom"
                                  >
                                    <input
                                      id="address"
                                      v-model="editForm.location.address"
                                      placeholder="Address"
                                      class="mt-2 form-input block w-full py-2 px-3 border border-gray-400 rounded-md shadow-sm focus:outline-none focus:shadow-outline-blue focus:border-blue-300 transition duration-150 ease-in-out sm:text-sm sm:leading-5"
                                    >
                                  </b-tooltip>
                                </div>
                                <div
                                  class="relative col-span-6 w-full flex flex-row items-center mt-2"
                                >
                                  <div
                                    class="w-full mb-2 flex flex-row appearance-none bg-white text-gray-600 border border-gray-400 hover:border-gray-500 px-4 py-2 pr-8 rounded shadow leading-tight focus:outline-none focus:shadow-outline"
                                    @click="showProductsModalEdit"
                                  >
                                    <span class="text-gray-600 font-medium">Select products</span>
                                    <span v-if="editForm.product_ids.length !== 0">({{ editForm.product_ids.length }})</span>
                                    <div
                                      class="absolute top-0 mt-2 right-0 px-2 text-gray-700 text-lg font-semibold"
                                    >
                                      <span>+</span>
                                    </div>
                                  </div>
                                  <b-modal
                                    :active.sync="showProductsEdit"
                                    has-modal-card
                                    trap-focus
                                    :destroy-on-hide="false"
                                    aria-role="dialog"
                                    aria-modal
                                  >
                                    <div class="modal-card">
                                      <div
                                        id="dropdown"
                                        class="w-full block p-3 shadow-md modal-card-body bg-gray-200"
                                      >
                                        <div class="flex flex-row w-full mb-2">
                                          <div class="w-full flex flex-col items-center">
                                            <span
                                              class="text-xl text-gray-600 font-semibold"
                                            >Select products</span>
                                          </div>
                                          <span
                                            class="rounded px-2 py-1 bg-red-700 text-white float-right cursor-pointer"
                                            @click="showProductsModalEdit"
                                          >Done</span>
                                        </div>
                                        <div class="flex flex-row justify-between">
                                          <label class="flex flex-row justify-start items-start">
                                            <div
                                              class="select-none cursor-pointer mr-1 uppercase text-sm font-semibold"
                                            >{{ selectAllText }}</div>
                                            <div
                                              class="bg-white border-2 rounded border-gray-800 w-6 h-6 flex flex-shrink-0 justify-center items-center mr-2 focus-within:border-blue-500"
                                            >
                                              <input
                                                id="selectAll"
                                                v-model="allSelected"
                                                type="checkbox"
                                                class="opacity-0 absolute cursor-pointer"
                                                @click="selectAll"
                                              >
                                              <svg
                                                class="fill-current hidden w-4 h-4 text-red-700 pointer-events-none"
                                                viewBox="0 0 20 20"
                                              >
                                                <path d="M0 11l2-2 5 5L18 3l2 2L7 18z" />
                                              </svg>
                                            </div>
                                          </label>
                                          <div
                                            class="text-sm font-semibold"
                                          >
                                            {{ editForm.product_ids.length }} SELECTED
                                          </div>
                                        </div>
                                        <ul id="customScroll" data-simplebar-auto-hide="false">
                                          <li
                                            v-for="(product, index) in products"
                                            :key="index"
                                            class="border border-t-0 border-r-0 border-l-0 border-b"
                                          >
                                            <label class="flex justify-start items-start">
                                              <div
                                                class="bg-white border-2 rounded border-gray-800 w-6 h-6 flex flex-shrink-0 justify-center items-center mr-2 focus-within:border-blue-500"
                                              >
                                                <input
                                                  :id="index"
                                                  v-model="editForm.product_ids"
                                                  type="checkbox"
                                                  :value="product.product_id"
                                                  class="opacity-0 absolute cursor-pointer"
                                                >
                                                <svg
                                                  class="fill-current hidden w-4 h-4 text-red-700 pointer-events-none"
                                                  viewBox="0 0 20 20"
                                                >
                                                  <path d="M0 11l2-2 5 5L18 3l2 2L7 18z" />
                                                </svg>
                                              </div>
                                              <div
                                                class="select-none cursor-pointer"
                                              >{{ product.name }}</div>
                                            </label>
                                          </li>
                                        </ul>
                                      </div>
                                    </div>
                                  </b-modal>
                                </div>
                                <div class="relative col-span-6 w-full flex flex-row items-center">
                                  <select
                                    v-model="editForm.day_of_week"
                                    class="block w-full appearance-none bg-white text-gray-600 border border-gray-400 hover:border-gray-500 px-4 py-2 pr-8 rounded shadow leading-tight focus:outline-none focus:shadow-outline"
                                  >
                                    <option disabled value>
                                      Day of week
                                    </option>
                                    <option>Monday</option>
                                    <option>Tuesday</option>
                                    <option>Wednesday</option>
                                    <option>Thursday</option>
                                    <option>Friday</option>
                                    <option>Saturday</option>
                                    <option>Sunday</option>
                                  </select>
                                  <div class="absolute top-0 mt-3 right-0 px-2 text-gray-700">
                                    <svg
                                      class="fill-current h-4 w-4"
                                      xmlns="http://www.w3.org/2000/svg"
                                      viewBox="0 0 20 20"
                                    >
                                      <path
                                        d="M9.293 12.95l.707.707L15.657 8l-1.414-1.414L10 10.828 5.757 6.586 4.343 8z"
                                      />
                                    </svg>
                                  </div>
                                </div>
                                <div class="col-span-6 flex flex-row justify-between px-3">
                                  <div class="w-full">
                                    <div class="flex flex-row">
                                      <b-tooltip
                                        label="Time when the location opens"
                                        class="w-full"
                                        type="is-info"
                                        position="is-top"
                                      >
                                        <input
                                          v-model="startEdit"
                                          type="range"
                                          name
                                          min="0"
                                          max="48"
                                          step="1"
                                          @input="getTimesOfDays(minVal, 'startEdit')"
                                        >
                                      </b-tooltip>
                                      <b-tooltip
                                        label="Time when the location closes"
                                        class="w-full"
                                        type="is-info"
                                        position="is-top"
                                      >
                                        <input
                                          v-model="endEdit"
                                          type="range"
                                          name
                                          min="0"
                                          max="48"
                                          step="1"
                                          @input="getTimesOfDays(maxVal, 'endEdit')"
                                        >
                                      </b-tooltip>
                                    </div>
                                    <div class="flex flex-row justify-between">
                                      <span>Start time: {{ editForm.start_time }}</span>
                                      <span>End time: {{ editForm.end_time }}</span>
                                    </div>
                                  </div>
                                </div>

                                <div v-if="error" class="w-full bg-red-200 p-2 rounded m-2">
                                  <span
                                    class="text-xs text-red-700 font-semibold font-ubuntu"
                                  >{{ errorMessage }}</span>
                                </div>
                                <div class="col-span-6 w-full mt-6 mb-2">
                                  <button
                                    class="group relative w-full flex justify-center py-2 px-4 border border-transparent text-sm leading-5 font-medium rounded-md text-white bg-gray-800 hover:bg-red-600 focus:outline-none focus:border-gray-700 focus:shadow-outline-gray active:bg-gray-600 transition duration-150 ease-in-out"
                                    @click.prevent="editPickup(pickupObj.pickup_id)"
                                  >
                                    <span v-if="isLoading">Updating...</span>
                                    <span v-else>Update pickup</span>
                                  </button>
                                </div>
                              </div>
                            </div>
                          </form>
                        </div>
                      </div>
                    </div>
                  </b-modal>
                </div>
              </client-only>
              <div class="w-full">
                <button
                  class="py-2 px-4 float-right shadow-xl border border-transparent text-sm leading-5 font-medium rounded-md text-white bg-gray-700 hover:bg-gray-600 focus:outline-none focus:shadow-outline-blue active:bg-gray-800 transition duration-150 ease-in-out"
                  @click="toggleState"
                >
                  <span class="text-base font-bold mr-2">+</span>
                  <span>Add new pickup</span>
                </button>
              </div>
            </div>
          </div>
          <div
            class="w-full flex flex-col items-center"
            :class="showPickupForm ? 'block' : 'hidden'"
          >
            <div class="w-full ml-2 md:absolute md:top-0 md:mt-2">
              <button
                class="py-2 px-4 float-left border border-transparent text-xl leading-5 font-semibold text-gray-700 focus:outline-none transition duration-150 ease-in-out"
                @click="toggleState"
              >
                <span class>
                  <font-awesome-icon :icon="['fa', 'arrow-left']" />
                </span>
              </button>
            </div>
            <div class="lg:w-3/4 md:w-3/4 lg:p-40 w-full p-5 lg:pt-5">
              <p class="flex flex-row row-gap-3 text-xl font-semibold mb-3">
                <img src="/plus.svg" alt class="h-6 w-6 mt-1 mr-2"> Create pickup locations
              </p>

              <div class="w-full flex flex-col bg-white rounded shadow-lg my-2 p-2">
                <div class="grid grid-cols-6 w-full">
                  <div class="col-span-6 w-full">
                    <b-tooltip
                      label="Name of pickup location (internal use)"
                      class="w-full"
                      type="is-info"
                      position="is-left"
                    >
                      <input
                        id="pickup-name"
                        v-model="form.name"
                        placeholder="Pickup name"
                        class="mt-2 form-input block w-full py-2 px-3 border border-gray-400 rounded-md shadow-sm focus:outline-none focus:shadow-outline-blue focus:border-blue-300 transition duration-150 ease-in-out sm:text-sm sm:leading-5"
                      >
                    </b-tooltip>
                  </div>
                  <div class="col-span-6 w-full">
                    <b-tooltip
                      label="The name displayed to your customers"
                      class="w-full"
                      type="is-info"
                      position="is-left"
                    >
                      <input
                        id="pickup-title"
                        v-model="form.title"
                        placeholder="Pickup title"
                        class="mt-2 form-input block w-full py-2 px-3 border border-gray-400 rounded-md shadow-sm focus:outline-none focus:shadow-outline-blue focus:border-blue-300 transition duration-150 ease-in-out sm:text-sm sm:leading-5"
                      >
                    </b-tooltip>
                  </div>
                  <div class="col-span-6 w-full">
                    <b-tooltip
                      label="Describe the location"
                      class="w-full"
                      type="is-info"
                      position="is-left"
                    >
                      <textarea
                        id="pickup-description"
                        v-model="form.description"
                        placeholder="Pickup description"
                        class="mt-2 form-input block w-full py-2 px-3 border border-gray-400 rounded-md shadow-sm focus:outline-none focus:shadow-outline-blue focus:border-blue-300 transition duration-150 ease-in-out sm:text-sm sm:leading-5"
                      />
                    </b-tooltip>
                  </div>
                  <div class="col-span-6 sm:col-span-3">
                    <b-tooltip
                      label="Zip/postcode of the location"
                      class="w-full"
                      type="is-info"
                      position="is-left"
                    >
                      <input
                        id="zip-code"
                        v-model="form.location.zip"
                        placeholder="Zip code"
                        class="mt-2 form-input block w-full py-2 px-3 border border-gray-400 rounded-md shadow-sm focus:outline-none focus:shadow-outline-blue focus:border-blue-300 transition duration-150 ease-in-out sm:text-sm sm:leading-5"
                      >
                    </b-tooltip>
                  </div>
                  <div class="col-span-6 sm:col-span-3">
                    <b-tooltip
                      label="City where pickup location is sited"
                      class="w-full"
                      type="is-info"
                      position="is-right"
                    >
                      <input
                        id="city"
                        v-model="form.location.city"
                        placeholder="City"
                        class="mt-2 ml-1 form-input block w-full py-2 px-3 border border-gray-400 rounded-md shadow-sm focus:outline-none focus:shadow-outline-blue focus:border-blue-300 transition duration-150 ease-in-out sm:text-sm sm:leading-5"
                      >
                    </b-tooltip>
                  </div>
                  <div class="col-span-6 w-full">
                    <b-tooltip
                      label="State where pickup location is sited"
                      class="w-full"
                      type="is-info"
                      position="is-left"
                    >
                      <input
                        id="state"
                        v-model="form.location.state"
                        placeholder="State"
                        class="mt-2 form-input block w-full py-2 px-3 border border-gray-400 rounded-md shadow-sm focus:outline-none focus:shadow-outline-blue focus:border-blue-300 transition duration-150 ease-in-out sm:text-sm sm:leading-5"
                      >
                    </b-tooltip>
                  </div>
                  <div class="col-span-6 w-full">
                    <b-tooltip
                      label="Country where pickup location is sited"
                      class="w-full"
                      type="is-info"
                      position="is-left"
                    >
                      <input
                        id="state"
                        v-model="form.location.country"
                        placeholder="Country"
                        class="mt-2 form-input block w-full py-2 px-3 border border-gray-400 rounded-md shadow-sm focus:outline-none focus:shadow-outline-blue focus:border-blue-300 transition duration-150 ease-in-out sm:text-sm sm:leading-5"
                      >
                    </b-tooltip>
                  </div>
                  <div class="col-span-6 w-full">
                    <b-tooltip
                      label="Full address of the pickup location"
                      class="w-full"
                      type="is-info"
                      position="is-left"
                    >
                      <input
                        id="address"
                        v-model="form.location.address"
                        placeholder="Address"
                        class="mt-2 form-input block w-full py-2 px-3 border border-gray-400 rounded-md shadow-sm focus:outline-none focus:shadow-outline-blue focus:border-blue-300 transition duration-150 ease-in-out sm:text-sm sm:leading-5"
                      >
                    </b-tooltip>
                  </div>
                  <div class="relative col-span-6 w-full flex flex-col items-center mt-2">
                    <div
                      class="w-full mb-2 flex flex-row appearance-none bg-white text-gray-600 border border-gray-400 hover:border-gray-500 px-4 py-2 pr-8 rounded shadow leading-tight focus:outline-none focus:shadow-outline"
                      @click="showProductsModal"
                    >
                      <span class="text-gray-600 font-medium">Select products</span>
                      <span v-if="form.product_ids.length !== 0">({{ form.product_ids.length }})</span>
                      <div
                        class="absolute top-0 mt-2 right-0 px-2 text-gray-700 text-lg font-semibold"
                      >
                        <span>+</span>
                      </div>
                    </div>
                    <b-modal
                      :active.sync="showProducts"
                      has-modal-card
                      trap-focus
                      :destroy-on-hide="false"
                      aria-role="dialog"
                      aria-modal
                    >
                      <div class="modal-card">
                        <div
                          id="dropdown"
                          class="w-full block p-3 shadow-md modal-card-body bg-gray-200"
                        >
                          <div class="flex flex-row w-full mb-2">
                            <div class="w-full flex flex-col items-center">
                              <span class="text-xl text-gray-600 font-semibold">Select products</span>
                            </div>
                            <span
                              class="rounded px-2 py-1 bg-red-700 text-white float-right cursor-pointer"
                              @click="showProductsModal"
                            >Done</span>
                          </div>
                          <div class="flex flex-row justify-between">
                            <label class="flex flex-row justify-start items-start">
                              <div
                                class="select-none cursor-pointer mr-1 uppercase text-sm font-semibold"
                              >{{ selectAllText }}</div>
                              <div
                                class="bg-white border-2 rounded border-gray-800 w-6 h-6 flex flex-shrink-0 justify-center items-center mr-2 focus-within:border-blue-500"
                              >
                                <input
                                  id="selectAll"
                                  v-model="allSelected"
                                  type="checkbox"
                                  class="opacity-0 absolute cursor-pointer"
                                  @click="selectAll"
                                >
                                <svg
                                  class="fill-current hidden w-4 h-4 text-red-700 pointer-events-none"
                                  viewBox="0 0 20 20"
                                >
                                  <path d="M0 11l2-2 5 5L18 3l2 2L7 18z" />
                                </svg>
                              </div>
                            </label>
                            <div
                              class="text-sm font-semibold"
                            >
                              {{ form.product_ids.length }} SELECTED
                            </div>
                          </div>
                          <ul id="customScroll" data-simplebar-auto-hide="false">
                            <li
                              v-for="(product, index) in products"
                              :key="index"
                              class="border border-t-0 border-r-0 border-l-0 border-b"
                            >
                              <label class="flex justify-start items-start">
                                <div
                                  class="bg-white border-2 rounded border-gray-800 w-6 h-6 flex flex-shrink-0 justify-center items-center mr-2 focus-within:border-blue-500"
                                >
                                  <input
                                    :id="index"
                                    v-model="form.product_ids"
                                    type="checkbox"
                                    :value="product.product_id"
                                    class="opacity-0 absolute cursor-pointer"
                                  >
                                  <svg
                                    class="fill-current hidden w-4 h-4 text-red-700 pointer-events-none"
                                    viewBox="0 0 20 20"
                                  >
                                    <path d="M0 11l2-2 5 5L18 3l2 2L7 18z" />
                                  </svg>
                                </div>
                                <div class="select-none cursor-pointer">{{ product.name }}</div>
                              </label>
                            </li>
                          </ul>
                        </div>
                      </div>
                    </b-modal>
                    <div class="relative col-span-6 w-full flex flex-row items-center">
                      <select
                        v-model="form.day_of_week"
                        class="block w-full appearance-none bg-white text-gray-600 border border-gray-400 hover:border-gray-500 px-4 py-2 pr-8 rounded shadow leading-tight focus:outline-none focus:shadow-outline"
                      >
                        <option disabled value>
                          Day of week
                        </option>
                        <option>Monday</option>
                        <option>Tuesday</option>
                        <option>Wednesday</option>
                        <option>Thursday</option>
                        <option>Friday</option>
                        <option>Saturday</option>
                        <option>Sunday</option>
                      </select>
                      <div class="absolute top-0 mt-3 right-0 px-2 text-gray-700">
                        <svg
                          class="fill-current h-4 w-4"
                          xmlns="http://www.w3.org/2000/svg"
                          viewBox="0 0 20 20"
                        >
                          <path
                            d="M9.293 12.95l.707.707L15.657 8l-1.414-1.414L10 10.828 5.757 6.586 4.343 8z"
                          />
                        </svg>
                      </div>
                    </div>
                    <div class="col-span-6 flex flex-row w-full justify-between px-3">
                      <div class="w-full">
                        <div class="flex flex-row">
                          <b-tooltip
                            label="Time when the location opens"
                            class="w-full"
                            type="is-info"
                            position="is-top"
                          >
                            <input
                              v-model="start"
                              type="range"
                              name
                              min="0"
                              max="48"
                              step="1"
                              @input="getTimesOfDays(minVal, 'start')"
                            >
                          </b-tooltip>
                          <b-tooltip
                            label="Time when the location closes"
                            class="w-full"
                            type="is-info"
                            position="is-top"
                          >
                            <input
                              v-model="end"
                              type="range"
                              name
                              min="0"
                              max="48"
                              step="1"
                              @input="getTimesOfDays(maxVal, 'end')"
                            >
                          </b-tooltip>
                        </div>
                        <div class="flex flex-row justify-between">
                          <span>Start time: {{ form.start_time }}</span>
                          <span>End time: {{ form.end_time }}</span>
                        </div>
                      </div>
                    </div>

                    <div v-if="error" class="col-span-6 w-full bg-red-200 p-2 rounded m-2">
                      <span
                        class="text-xs text-red-700 font-semibold font-ubuntu"
                      >{{ errorMessage }}</span>
                    </div>
                    <div class="col-span-6 w-full py-3">
                      <button
                        :disabled="incomplete"
                        class="w-full py-2 px-4 shadow-xl border border-transparent text-sm leading-5 font-medium rounded-md text-white bg-red-700 disabled:cursor-not-allowed hover:bg-red-600 focus:outline-none focus:shadow-outline-blue active:bg-gray-800 transition duration-150 ease-in-out"
                        @click="addPickup"
                      >
                        <span v-if="isLoading">Adding pickup...</span>
                        <span v-else>Add pickup</span>
                      </button>
                    </div>
                  </div>
                </div>
              </div>
            </div>
          </div>
        </div>
      </main>
    </div>
  </div>
</template>

<script>
import { mapState } from 'vuex';
import Sidenav from '../../components/sidenav';
import Paginator from '../../components/pagination';
export default {
  layout: 'none',
  components: {
    Sidenav,
    Paginator
  },
  data () {
    return {
      dropDownStateId: '',
      hoursOfDays: this.getTimesOfDays(),
      isLoading: false,
      incomplete: false,
      showPickupForm: false,
      allSelected: false,
      selectAllText: 'Select All',
      showProducts: false,
      showProductsEdit: false,
      makeModal: false,
      current_page: 1,
      limit: 10,
      products: {},
      form: {
        name: '',
        title: '',
        description: '',
        location: {
          zip: '',
          state: '',
          city: '',
          country: '',
          address: ''
        },
        store_id: '',
        product_ids: [],
        day_of_week: '',
        start_time: '',
        end_time: ''
      },
      editForm: {
        name: '',
        title: '',
        description: '',
        location: {
          zip: '',
          state: '',
          city: '',
          country: '',
          address: ''
        },
        store_id: '',
        product_ids: [],
        day_of_week: '',
        start_time: '',
        end_time: '',
        updated_at: ''
      },
      minVal: '',
      maxVal: '',
      product_id: '',
      pickupData: [],
      pickupObj: {},
      error: false,
      errorMessage: '',
      newList: []
    };
  },

  computed: {
    ...mapState(['token', 'store', 'pickups']),
    start: {
      set (val) {
        this.minVal = val;
      },

      get () {
        return this.minVal;
      }
    },

    startEdit: {
      set (val) {
        this.minVal = val;
      },

      get () {
        return this.minVal;
      }
    },

    end: {
      set (val) {
        this.maxVal = val;
      },

      get () {
        return this.maxVal;
      }
    },

    endEdit: {
      set (val) {
        this.maxVal = val;
      },

      get () {
        return this.maxVal;
      }
    }
  },

  async created () {
    const data = await this.$axios.get(
      `/stores/pickups/${this.store.store_id}`,
      {
        headers: {
          'x-access-token': this.token
        }
      }
    );

    this.pickupData = data.data.pickups;

    const productResponse = await this.$axios.get(
      `/onboarding/stores/${this.store.store_id}/products`,
      {
        headers: {
          'x-access-token': this.token
        }
      }
    );

    this.products = productResponse.data.products;

    await this.$axios
      .get(
        `/onboarding/stores/${this.store.store_id}/pickups?limit=${this.limit}&offset=0`,
        {
          headers: {
            'x-access-token': this.token
          }
        }
      )
      .then((pickups) => {
        this.$store.commit('setPickups', pickups.data.pickups);
      })
      .catch(() => {
        return this.$buefy.notification.open({
          duration: 2000,
          closable: false,
          animation: 'fade',
          message: "There's an error fetching pickups at this time.",
          position: 'is-bottom-right',
          type: 'is-primary'
        });
      });
    this.form.store_id = this.store.store_id;
  },

  methods: {
    toggleDropDown (id) {
      this.dropDownStateId === '' ? this.dropDownStateId = id : this.dropDownStateId = '';
    },

    getCurrentTopAndBottomItem () {
      const len = this.pickups.length;
      const top = this.pickups[len - len].ui_order;
      const bottom = this.pickups[len - 1].ui_order;

      return { top, bottom };
    },

    async moveItemTo (position, pickup_id) {
      try {
        const { top, bottom } = this.getCurrentTopAndBottomItem();
        await this.$axios.put(
          `/onboarding/stores/${this.store.store_id}/pickups/${pickup_id}`,
          { ui_order: position == 'top' ? top + 1 : bottom - 1 },
          {
            headers: {
              'x-access-token': this.token
            }
          }
        );

        // update localStorage after updating item
        this.updateLocalStorage();
        this.dropDownStateId = '';
      } catch (error) {
        console.error(error);
        return this.$buefy.notification.open({
          duration: 2000,
          closable: false,
          animation: 'fade',
          message: 'Error: unable to move item.',
          position: 'is-bottom-right',
          type: 'is-primary'
        });
      }
    },

    async updateLocalStorage () {
      const pickups = await this.$axios.get(
        `/onboarding/stores/${this.store.store_id}/pickups?limit=${
          this.limit
        }&offset=${(this.current_page - 1) * this.limit}`,
        {
          headers: {
            'x-access-token': this.token
          }
        }
      );
      this.$store.commit('setPickups', pickups.data.pickups);
    },

    getTimesOfDays (val, range) {
      const items = [];
      for (let hour = 0; hour <= 24; hour++) {
        items.push([hour, 0]);
        items.push([hour, 30]);
      }

      const date = new Date();
      const formatter = new Intl.DateTimeFormat('en-US', {
        hour: 'numeric',
        minute: 'numeric'
      });

      items.forEach((time, index) => {
        const [hour, minute] = time;
        date.setHours(hour);
        date.setMinutes(minute);
        const formattedDate = formatter.format(date);

        if (parseInt(val) === index) {
          switch (range) {
            case 'start':
              this.form.start_time = formattedDate;
              break;
            case 'end':
              this.form.end_time = formattedDate;
              break;
            case 'startEdit':
              this.editForm.start_time = formattedDate;
              break;
            case 'endEdit':
              this.editForm.end_time = formattedDate;
              break;
            default:
              console.log('invalid');
              break;
          }
        }
      });
    },

    toggleState () {
      this.showPickupForm = !this.showPickupForm;
    },

    validateFormFields () {
      if (this.form.name === '' ||
          this.form.title === '' ||
          this.form.description === '' ||
          this.form.location.zip === '' ||
          this.form.location.address === '' ||
          this.form.location.state === '' ||
          this.form.location.country === '' ||
          this.form.location.city === ''
      ) {
        this.error = true;
        this.errorMessage = 'All form fields are required';
        this.isLoading = false;
        return false;
      }

      return true;
    },

    async openModal (pickup_id) {
      let pickup = await this.$axios.get(`/shop/pickups/${pickup_id}`, {
        headers: {
          'x-access-token': this.token
        }
      });
      pickup = pickup.data.pickup;
      this.makeModal = true;
      this.pickupObj = pickup;
      this.editForm.name = pickup.name;
      this.editForm.description = pickup.description;
      this.editForm.title = pickup.title;
      this.editForm.store_id = pickup.store_id;
      this.editForm.location.zip = pickup.location.zip;
      this.editForm.location.address = pickup.location.address;
      this.editForm.location.state = pickup.location.state;
      this.editForm.location.country = pickup.location.country;
      this.editForm.location.city = pickup.location.city;
      this.editForm.start_time = pickup.start_time;
      this.editForm.end_time = pickup.end_time;
      this.editForm.day_of_week = pickup.day_of_week.charAt(0).toUpperCase() + pickup.day_of_week.slice(1);
      this.editForm.product_ids = pickup.product_ids;
    },

    async editPickup (pickup_id) {
      this.isLoading = true;
      const date = new Date();
      try {
        this.editForm.updated_at = date.toISOString();
        this.editForm.day_of_week = String(
          this.editForm.day_of_week
        ).toLowerCase();

        // set pickup to all products if no product is set
        if (this.editForm.product_ids.length < 1) {
          this.products.forEach((item) => {
            this.editForm.product_ids.push(item.product_id);
          });
        }

        await this.$axios.put(
          `/onboarding/stores/${this.store.store_id}/pickups/${pickup_id}`,
          this.editForm,
          {
            headers: {
              'x-access-token': this.token
            }
          }
        );

        // update localStorage after updating item
        this.updateLocalStorage();
        this.makeModal = false;
        this.isLoading = false;
        this.maxVal = '';
        this.minVal = '';
      } catch (error) {
        console.log(error);
        this.isLoading = false;

        return this.$buefy.notification.open({
          duration: 2000,
          closable: false,
          animation: 'fade',
          message: 'unable to update pickup.',
          position: 'is-bottom-right',
          type: 'is-primary'
        });
      }
    },

    async deletePickup (pickup_id) {
      try {
        // delete single record from database
        await this.$axios.delete(
          `/onboarding/stores/${this.store.store_id}/pickup/${pickup_id}`,
          {
            headers: {
              'x-access-token': this.token
            }
          }
        );

        // update localStorage after deleting item
        this.updateLocalStorage();
      } catch {
        return this.$buefy.notification.open({
          duration: 2000,
          closable: false,
          animation: 'fade',
          message: 'unable to delete product.',
          position: 'is-bottom-right',
          type: 'is-primary'
        });
      }
    },

    async refetchPickups (page) {
      await this.$axios
        .get(
          `/onboarding/stores/${this.store.store_id}/pickups?limit=${
            this.limit
          }&offset=${(page - 1) * this.limit}`,
          {
            headers: {
              'x-access-token': this.token
            }
          }
        )
        .then((pickups) => {
          this.current_page = page;
          this.$store.commit('setPickups', pickups.data.pickups);
        })
        .catch(() => {
          return this.$buefy.notification.open({
            duration: 2000,
            closable: false,
            animation: 'fade',
            message: "There's an error fetching pickups at this time.",
            position: 'is-bottom-right',
            type: 'is-primary'
          });
        });
    },

    async addPickup () {
      this.isLoading = true;
      const validate = this.validateFormFields();

      if (validate) {
        try {
        // set pickup to all products if no product is set
          if (this.form.product_ids.length < 1) {
            this.products.forEach((item) => {
              this.form.product_ids.push(item.product_id);
            });
          }

          this.form.day_of_week = String(this.form.day_of_week).toLowerCase();
          await this.$axios.post(
          `/onboarding/stores/${this.store.store_id}/pickups`,
          this.form,
          {
            headers: {
              'x-access-token': this.token
            }
          }
          );

          // update localStorage after updating item
          this.updateLocalStorage();

          this.isLoading = false;
          this.error = false;
          return this.$buefy.notification.open({
            duration: 2000,
            closable: false,
            animation: 'fade',
            message: 'Pickup created successfully.',
            position: 'is-bottom-right',
            type: 'is-success'
          });
        } catch (error) {
          this.error = true;
          this.errorMessage = 'An unexpected error occurred';
          this.isLoading = false;
          console.log(error);
        } finally {
          this.form.name = '';
          this.form.title = '';
          this.form.description = '';
          this.form.day_of_week = '';
          this.form.start_time = '';
          this.form.end_time = '';
          this.form.location.zip = '';
          this.form.location.city = '';
          this.form.location.state = '';
          this.form.location.address = '';
          this.form.location.country = '';
          this.form.product_ids = [];
          this.maxVal = '';
          this.minVal = '';
        }
      }
    },

    showProductsModal () {
      this.showProducts = !this.showProducts;
    },

    showProductsModalEdit () {
      this.showProductsEdit = !this.showProductsEdit;
    },

    selectAll () {
      this.allSelected = !this.allSelected;
      this.selectAllText =
        this.selectAllText == 'Select All' ? 'Clear All' : 'Select All';
      if (this.showProductsEdit) {
        if (this.allSelected) {
          for (const item in this.products) {
            this.editForm.product_ids.push(this.products[item].product_id);
          }
        } else {
          this.editForm.product_ids = [];
        }
      }

      if (this.showProducts) {
        if (this.allSelected) {
          for (const item in this.products) {
            this.form.product_ids.push(this.products[item].product_id);
          }
        } else {
          this.form.product_ids = [];
        }
      }
    }
  }
};
</script>

<style>
input:checked + svg {
  display: block;
}
</style>

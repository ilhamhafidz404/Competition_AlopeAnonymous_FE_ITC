<template>
  <section
    class="grid lg:grid-cols-3 md:grid-cols-2 mb-16 mt-28 mx-5 sm:mx-10 lg:mx-24 gap-5 font-primary"
  >
    <div>
      <div class="border border-red-500 rounded-[15px] shadow-xl">
        <img
          :src="'https://alope.id/storage/' + volunteer.image_url"
          alt=""
          class="w-full rounded-[15px]"
        />
        <div class="my-7 mx-2 text-center">
          <span
            class="bg-red-500/20 text-red-500 font-semibold px-5 py-3 rounded text-lg mx-1 mb-2 inline-block"
          >
            {{ volunteer.category }}
          </span>
        </div>
      </div>
    </div>
    <div class="lg:col-span-2 border border-primary rounded-[15px] shadow-xl">
      <div class="md:my-5 md:mx-10 my-3 mx-5">
        <button
          class="flex items-center mb-6 text-red-500 font-bold hover:text-primary transition duration-300"
          @click="goToHome"
        >
          <i class="fas fa-arrow-left mr-2"></i> Kembali
        </button>

        <!-- Deskripsi Program -->
        <h1 class="md:text-4xl text-3xl font-bold">{{ volunteer.title }}</h1>
        <h2 class="md:text-3xl text-2xl font-semibold mt-5 text-red-500">
          Deskripsi
        </h2>
        <p class="text-gray-700 my-5 text-lg">
          {{ volunteer.description }}
        </p>
        <h2 class="md:text-3xl text-2xl font-semibold mt-5 text-red-500">
          Waktu
        </h2>
        <div class="flex items-center my-3 gap-3">
          <i
            class="fa-regular fa-calendar-days md:text-2xl text-lg rounded-lg p-2 text-gray-700"
          ></i>
          <p href="" class="text-lg text-gray-700">
            {{ formatDate(volunteer.start_date) }} <span> - </span>
            {{ formatDate(volunteer.end_date) }}
          </p>
        </div>
        <h2 class="md:text-3xl text-2xl font-semibold mt-5 text-red-500">
          Contact Us
        </h2>
        <div class="flex items-center mt-2 gap-3">
          <i
            class="fa-brands fa-instagram md:text-2xl text-lg rounded-lg p-2 text-gray-700"
          ></i>
          <p href="" class="text-lg text-gray-700">
            {{ volunteer.contact_instagram }}
          </p>
        </div>
        <div class="flex items-center mb-3 gap-3">
          <i
            class="fa-solid fa-phone md:text-2xl text-lg rounded-lg p-2 text-gray-700"
          ></i>
          <p href="" class="text-lg text-gray-700">
            {{ volunteer.contact_phone }}
          </p>
        </div>
        <button
          v-if="isLoggedIn"
          @click="getDaftarVolunteer"
          class="border border-red-500 text-red-500 px-4 py-2 mt-4 rounded-md font-bold hover:bg-red-500 hover:text-white transition duration-300"
        >
          Daftar Sekarang
        </button>
        <button
          v-else
          @click="promptLogin"
          class="border border-red-500 text-red-500 px-4 py-2 mt-4 rounded-md font-bold hover:bg-red-500 hover:text-white transition duration-300"
        >
          Daftar Sekarang
        </button>
      </div>
    </div>
  </section>
</template>

<script>
import axios from "axios";
import { RouterLink } from "vue-router";
import moment from "moment";
import { data } from "autoprefixer";

export default {
  props: {
    id: {
      type: [String, Number],
      required: true,
    },
  },
  data() {
    return {
      isLoadingGetVolunteer: false,
      volunteer: {},
      error: null,
      isLoggedIn: false,
      idUser: 0,
    };
  },
  methods: {
    getDaftarVolunteer() {
      const token = localStorage.getItem("userToken");
      if (!token) {
        console.error("Token not found in localStorage");
        return;
      }

      // Header dengan Authorization
      const headers = {
        Authorization: `Bearer ${token}`,
      };

      axios
        .post(
          "https://alope.id/api/user/volunteerAPI/register",
          {
            volunteer_id: this.id,
          },
          { headers }
        )
        .then((response) => {
          this.$router.push({ name: "profile" });
          window.open(this.volunteer.registration_url, "_blank");

          console.log(this.volunteer.registration_url); 
        })
        .catch((error) => {
          console.error("Server error:", error);
        });
    },

    getDataProfile() {
      // Ambil token dari localStorage
      const token = localStorage.getItem("userToken"); // Ganti "token" dengan nama key yang digunakan saat menyimpan token
      if (!token) {
        console.error("Token not found in localStorage");
        return;
      }

      // Header dengan Authorization
      const headers = {
        Authorization: `Bearer ${token}`,
      };

      axios
        .get("https://alope.id/api/user/profile", { headers }) // Tambahkan headers di sini
        .then((response) => {
          if (response && response.data) {
            this.idUser = response.data.data.id;
            console.log(this.idUser);
          }
        })
        .catch((error) => {
          console.error("Server error:", error);
        });
    },
    formatDate(date) {
      return moment(date).format("DD MMMM YYYY");
    },
    async getDataVolunteer() {
      try {
        this.isLoadingGetVolunteer = true;
        this.error = null;

        // Memanggil API dengan ID
        const response = await axios.get(
          `https://alope.id/api/user/volunteerAPI/${this.id}`
        );

        if (response && response.data) {
          this.volunteer = response.data.data;
          console.log(this.id);
        }
      } catch (error) {
        console.error("Server error:", error);
        this.error = "Gagal mengambil data. Silakan coba lagi.";
      } finally {
        this.isLoadingGetVolunteer = false;
      }
    },
    promptLogin() {
      alert("Harap login terlebih dahulu untuk mendaftar!");
      this.$router.push({ name: "login" });
    },
    checkLoginStatus() {
      const token = localStorage.getItem("userToken");
      this.isLoggedIn = !!token;
    },
    goToHome() {
      this.$router.push({ name: "home" });
    },
  },
  mounted() {
    this.getDataVolunteer();
    this.checkLoginStatus();
    this.getDataProfile();
    window.scrollTo(0, 0);
  },
};
</script>

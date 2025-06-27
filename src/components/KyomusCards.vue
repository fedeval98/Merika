<script setup>
import kyomus from "../assets/kyomus.json";
import axios from "axios";
import { reactive, onMounted } from "vue";

// Estado reactivo para saber quién está online
const onlineStatus = reactive({});

// Twitch API
const clientId = "dimqlj95z01r9sfao2jd4t70qem69n";
const clientSecret = "zgw1ilrznrk17gaxk857w3yk1c47wj";
const tokenUrl = "https://id.twitch.tv/oauth2/token";

// Obtener accessToken
async function getAccessToken() {
  const formData = new URLSearchParams();
  formData.append("client_id", clientId);
  formData.append("client_secret", clientSecret);
  formData.append("grant_type", "client_credentials");

  const response = await axios.post(tokenUrl, formData, {
    headers: { "Content-Type": "application/x-www-form-urlencoded" },
  });

  return response.data.access_token;
}

// Verificar si está online
async function checkIfOnline(broadcasterId, accessToken) {
  const response = await axios.get(
    `https://api.twitch.tv/helix/streams?user_id=${broadcasterId}`,
    {
      headers: {
        Authorization: `Bearer ${accessToken}`,
        "Client-Id": clientId,
      },
    }
  );

  return !!response.data.data[0];
}

// Cambiar imagen entre on/off usando URLs del JSON
function toggleImgSrc(img, kyomu, toState = "off") {
  if (!kyomu?.img) return;
  img.src = toState === "on" ? kyomu.img.on : kyomu.img.off;
}

// Cargar estados y configurar interacciones
onMounted(async () => {
  const accessToken = await getAccessToken();
  const kyomuList = Object.values(kyomus);

  for (const kyomu of kyomuList) {
    const status = await checkIfOnline(kyomu.id, accessToken);
    onlineStatus[kyomu.id] = status;
    console.log(`${kyomu.name} está ${status ? "ONLINE" : "OFFLINE"}`);
  }

  const listItems = document.querySelectorAll("ul li");

  listItems.forEach((li) => {
    const img = li.querySelector("img");
    const h3 = li.querySelector("h3");
    if (!img || !h3) return;

    li.addEventListener("mouseenter", () => {
      const name = h3.textContent.trim();
      const kyomu = kyomuList.find((k) => k.name === name);
      const color = kyomu?.color || "rgba(255, 255, 255, 0.3)";

      img.style.transition = "opacity 0.150s ease";
      img.style.opacity = 0;

      setTimeout(() => {
        h3.style.color = color;
        toggleImgSrc(img, kyomu, "on");
        img.classList.remove("grayscale");
        img.style.boxShadow = `${color} 0px -10px 80px 10px`;
        img.style.opacity = 1;
      }, 150);
    });

    li.addEventListener("mouseleave", () => {
      const name = h3.textContent.trim();
      const kyomu = kyomuList.find((k) => k.name === name);

      img.style.transition = "opacity 0.150s ease";
      img.style.opacity = 0;

      setTimeout(() => {
        h3.style.color = "white";
        toggleImgSrc(img, kyomu, "off");
        img.classList.add("grayscale");
        img.style.boxShadow = "none";
        img.style.opacity = 1;
      }, 150);
    });
  });
});
console.log(kyomus)
</script>


<template>
  <ul class="text-black flex flex-wrap gap-5">
    <!-- CARDS -->
    <template v-if="kyomus.length != 0" v-for="kyomu in kyomus">
      <li class="border p-4 bg-black text-white rounded-2xl w-80 h-80">
        <a href="https://linktr.ee/iamnolei">
          <figure class="flex justify-center items-center">
            <img :src="kyomu.img.off" class="grayscale w-56 h-56" alt="" />
          </figure>
          <h3 class="text-center font-bold">{{ kyomu.name }}</h3>
          <div>
            <h2
              v-if="onlineStatus[kyomu.id]"
              class="bg-green-500 px-4 text-lg lg:text-2xl font-mono text-white font-bold rounded-full text-center"
            >
              EN DIRECTO
            </h2>
            <h2
              v-else
              class="bg-red-500 px-4 text-lg lg:text-2xl font-mono text-white font-bold rounded-full text-center"
            >
              OFFLINE
            </h2>
          </div>
        </a>
      </li>
    </template>
  </ul>
</template>

<style scoped>
/* CLASE PARA DAR RESPLANDOR */
.selected-kyomu {
  box-shadow: rgba(241, 234, 98, 0.56) 0px -10px 80px 10px;
}
</style>

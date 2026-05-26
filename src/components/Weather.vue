<template>
  <div class="weather" v-if="weatherData.adCode.city && weatherData.weather.weather">
    <span>{{ weatherData.adCode.city }}&nbsp;</span>
    <span>{{ weatherData.weather.weather }}&nbsp;</span>
    <span>{{ weatherData.weather.temperature }}℃</span>
    <span class="sm-hidden">
      &nbsp;{{
        weatherData.weather.winddirection?.endsWith("风")
          ? weatherData.weather.winddirection
          : weatherData.weather.winddirection + "风"
      }}&nbsp;
    </span>
    <span class="sm-hidden">
      {{   
      weatherData.weather.windpower?.endsWith("级")   
        ? weatherData.weather.windpower   
        : weatherData.weather.windpower + "级"   
    }}&nbsp;
  </span>
  </div>
  <div class="weather" v-else>
    <span>这里应该显示天气</span>
  </div>
</template>

<script setup>
import { getAdcode, getWeather, getVvhanWeather } from "@/api";

const mainKey = import.meta.env.VITE_WEATHER_KEY;
const CACHE_KEY = "weather_cache";

const weatherData = reactive({
  adCode: { city: null, adcode: null },
  weather: { weather: null, temperature: null, winddirection: null, windpower: null },
});

// 读取缓存天气
const loadCache = () => {
  try {
    const raw = localStorage.getItem(CACHE_KEY);
    if (!raw) return false;
    const data = JSON.parse(raw);
    setWeather(data.city, data.adcode, data.weather, data.temperature, data.winddirection, data.windpower);
    return true;
  } catch { return false; }
};

// 写入缓存
const saveCache = () => {
  localStorage.setItem(CACHE_KEY, JSON.stringify({
    city: weatherData.adCode.city,
    adcode: weatherData.adCode.adcode,
    weather: weatherData.weather.weather,
    temperature: weatherData.weather.temperature,
    winddirection: weatherData.weather.winddirection,
    windpower: weatherData.weather.windpower,
  }));
};

// 取出天气平均值
const getTemperature = (min, max) => {
  try {
    return Math.round((Number(min) + Number(max)) / 2);
  } catch {
    return "NaN";
  }
};

// 设置天气数据
const setWeather = (city, adcode, weather, temperature, winddirection, windpower) => {
  weatherData.adCode = { city, adcode };
  weatherData.weather = { weather, temperature, winddirection, windpower };
};

// 请求天气 API 并写入 weatherData
const fetchAndSetWeather = async (adcode) => {
  const result = await getWeather(mainKey, adcode);
  if (!result.lives || result.lives.length === 0) return false;
  setWeather(
    null, adcode,
    result.lives[0].weather,
    result.lives[0].temperature,
    result.lives[0].winddirection,
    result.lives[0].windpower,
  );
  return true;
};

// 获取天气数据
const getWeatherData = async () => {
  try {
    // 无高德 Key：走备用 API
    if (!mainKey) {
      const result = await getVvhanWeather();
      if (!result.success) throw result.message || "备用天气API调用失败";
      const data = result.data;
      setWeather(
        result.city || "未知地区", null,
        data.type,
        getTemperature(data.low.replace("°C", ""), data.high.replace("°C", "")),
        data.fengxiang, data.fengli,
      );
      saveCache();
      return;
    }

    // 第一步：尝试 IP 定位天气
    let success = false;
    try {
      const adCode = await getAdcode(mainKey);
      if (adCode.infocode === "10000") {
        const ok = await fetchAndSetWeather(adCode.adcode);
        if (ok) {
          weatherData.adCode.city = adCode.city;
          weatherData.adCode.adcode = adCode.adcode;
          success = true;
        }
      }
    } catch (e) {
      console.error("IP定位天气失败:" + e);
    }

    // 第二步：IP 失败则请求上海天气
    if (!success) {
      const shanghaiOk = await fetchAndSetWeather("310000");
      if (!shanghaiOk) throw "上海天气为空";
      weatherData.adCode.city = "上海";
    }

    saveCache();
  } catch (error) {
    console.error("天气信息获取失败:" + error);
  }
};

// 先显示缓存，再后台更新
loadCache();
getWeatherData();
</script>

<!DOCTYPE html>
<html lang="bn">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>আজকের আবহাওয়া</title>
<style>
  body {
    font-family: "Noto Sans Bengali","Hind Siliguri",sans-serif;
    margin: 0;
    padding: 0;
    display: flex;
    justify-content: center;
    align-items: center;
    min-height: 100vh;
    transition: background 1s ease;
  }
  .weather-card {
    width: 95%;
    max-width: 420px;
    background: rgba(255,255,255,0.25);
    border-radius: 18px;
    box-shadow: 0 4px 10px rgba(0,0,0,0.2);
    backdrop-filter: blur(6px);
    padding: 10px 12px;
    text-align: center;
    color: #111;
  }
  h2 {
    margin: 4px 0;
    font-size: 20px;
  }
  .location {
    font-size: 16px;
    margin-bottom: 6px;
    color: #222;
  }
  .temp {
    font-size: 52px;
    font-weight: bold;
    margin: 5px 0;
  }
  .icon {
    font-size: 36px;
    margin-bottom: 6px;
  }
  .details {
    font-size: 15px;
    margin: 5px 0;
  }
  .forecast {
    display: grid;
    grid-template-columns: repeat(auto-fit,minmax(90px,1fr));
    gap: 6px;
    margin-top: 10px;
  }
  .day {
    background: rgba(255,255,255,0.65);
    border-radius: 10px;
    padding: 8px 4px;
    box-shadow: 0 1px 4px rgba(0,0,0,0.1);
  }
  .day .icon {
    font-size: 24px;
  }
  .small {
    font-size: 12px;
    color: #333;
  }
</style>
</head>
<body>
  <div class="weather-card" id="weatherCard">
    <h2>আজকের আবহাওয়া</h2>
    <div class="location" id="loc">অবস্থান নির্ণয় হচ্ছে...</div>
    <div class="temp" id="temp">--°C</div>
    <div class="icon" id="icon">☀️</div>
    <div class="details" id="desc">--</div>
    <div class="details" id="extra">--</div>
    <div class="forecast" id="forecast">৭ দিনের পূর্বাভাস লোড হচ্ছে...</div>
  </div>

<script>
async function getLocation() {
  return new Promise((resolve) => {
    if (navigator.geolocation) {
      navigator.geolocation.getCurrentPosition(
        pos => resolve({lat: pos.coords.latitude, lon: pos.coords.longitude, method:"gps"}),
        () => fetchIPLocation(resolve)
      );
    } else {
      fetchIPLocation(resolve);
    }
  });
}
async function fetchIPLocation(resolve){
  try {
    const res = await fetch("https://ipapi.co/json/");
    const data = await res.json();
    let city = data.city || await reverseGeocode(data.latitude,data.longitude);
    resolve({lat:data.latitude, lon:data.longitude, city:city || "অজানা স্থান", method:"ip"});
  } catch {
    resolve({lat:22.57, lon:88.36, city:"Kolkata", method:"fallback"});
  }
}
async function reverseGeocode(lat,lon){
  try{
    const res = await fetch(`https://geocode.maps.co/reverse?lat=${lat}&lon=${lon}`);
    const data = await res.json();
    return data.address.city || data.address.town || data.address.village || data.display_name.split(",")[0];
  }catch{return null;}
}

async function loadWeather() {
  const card=document.getElementById("weatherCard"),
        locElem=document.getElementById("loc"),
        tempElem=document.getElementById("temp"),
        iconElem=document.getElementById("icon"),
        descElem=document.getElementById("desc"),
        extraElem=document.getElementById("extra"),
        forecastElem=document.getElementById("forecast");

  const loc=await getLocation();
  const url=`https://api.open-meteo.com/v1/forecast?latitude=${loc.lat}&longitude=${loc.lon}&current=temperature_2m,relative_humidity_2m,precipitation,weathercode,wind_speed_10m&daily=weathercode,temperature_2m_max,temperature_2m_min,precipitation_sum&timezone=auto`;
  const res=await fetch(url);
  const data=await res.json();

  const c=data.current;
  const d=data.daily;
  const code=c.weathercode;

  const iconMap={0:"☀️",1:"🌤️",2:"⛅",3:"☁️",45:"🌫️",48:"🌫️",51:"🌦️",61:"🌧️",63:"🌧️",65:"🌧️",71:"❄️",73:"❄️",75:"❄️",95:"⛈️",99:"🌩️"};
  const bgMap={0:"#b8e0ff",1:"#c2e3ff",2:"#c9dbf4",3:"#d7d7d7",61:"#a0b6ff",95:"#324a7d",99:"#223259"};
  const grad=(bgMap[code])?`linear-gradient(to bottom,${bgMap[code]},#ffffff)`:"linear-gradient(to bottom,#d8ecff,#ffffff)";
  document.body.style.background=grad;

  locElem.textContent=loc.city || "অবস্থান শনাক্ত হয়েছে";
  tempElem.textContent=`${c.temperature_2m}°C`;
  iconElem.textContent=iconMap[code]||"☀️";
  descElem.textContent=getDesc(code);
  extraElem.textContent=`আর্দ্রতা: ${c.relative_humidity_2m}% • বাতাস: ${c.wind_speed_10m} km/h • বৃষ্টি: ${c.precipitation} mm`;

  let fHTML="";
  for(let i=0;i<7;i++){
    const dayName=new Date(d.time[i]).toLocaleDateString('bn-BD',{weekday:'short'});
    const ic=iconMap[d.weathercode[i]]||"☀️";
    fHTML+=`<div class="day"><div class="icon">${ic}</div>${dayName}<br><span class="small">${d.temperature_2m_max[i]}° / ${d.temperature_2m_min[i]}°</span></div>`;
  }
  forecastElem.innerHTML=fHTML;
}
function getDesc(code){
  const t={0:"আকাশ পরিষ্কার",1:"আংশিক মেঘলা",2:"মেঘাচ্ছন্ন",3:"পুরোপুরি মেঘাচ্ছন্ন",61:"বৃষ্টি",95:"বজ্রসহ বৃষ্টি",99:"ঝড়ো বৃষ্টি"};
  return t[code]||"স্বাভাবিক আবহাওয়া";
}
loadWeather();
</script>
</body>
</html>

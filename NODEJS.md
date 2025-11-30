# Node.js Intervyusidagi Eng Ko'p Beriladigan Savollar va Javoblar

## 1. **Node.js nima va u qanday ishlaydi?**

Node.js - bu Chrome V8 JavaScript mexanizmasi asosida qurilgan server-side JavaScript runtime muhiti. U event-driven, non-blocking I/O modelidan foydalanadi va bitta threadda ishlaydi, ammo event loop orqali ko'plab bir vaqtdagi operatsiyalarni boshqaradi.

**Asosiy xususiyatlari:**
- Asinxron va event-driven arxitektura
- NPM (Node Package Manager) orqali katta kutubxonalar ekosistemi
- Yuqori unumdorlik va masshtablanish imkoniyati
- Real-time ilovalar uchun ideal

## 2. **Event Loop qanday ishlaydi?**

Event Loop - Node.js'ning asosiy komponentidir. U asinxron operatsiyalarni boshqaradi va quyidagi fazalardan iborat:

- **Timers**: setTimeout va setInterval callback'larini bajaradi
- **Pending callbacks**: I/O operatsiyalarining callback'lari
- **Idle, prepare**: Ichki foydalanish uchun
- **Poll**: Yangi I/O eventlarini oladi
- **Check**: setImmediate callback'lari
- **Close callbacks**: close eventlari

## 3. **Callback, Promise va Async/Await o'rtasidagi farq nima?**

**Callback:**
```javascript
fs.readFile('file.txt', (err, data) => {
  if (err) throw err;
  console.log(data);
});
```

**Promise:**
```javascript
readFilePromise('file.txt')
  .then(data => console.log(data))
  .catch(err => console.error(err));
```

**Async/Await:**
```javascript
async function readFile() {
  try {
    const data = await readFilePromise('file.txt');
    console.log(data);
  } catch (err) {
    console.error(err);
  }
}
```

## 4. **Blocking vs Non-blocking code nima?**

**Blocking (Sinxron):**
```javascript
const data = fs.readFileSync('file.txt');
console.log(data);
// Keyingi kod kutadi
```

**Non-blocking (Asinxron):**
```javascript
fs.readFile('file.txt', (err, data) => {
  console.log(data);
});
// Keyingi kod darhol bajariladi
```

## 5. **Middleware nima?**

Middleware - bu request va response o'rtasida bajariladigan funksiyalar. Express.js'da keng qo'llaniladi:

```javascript
app.use((req, res, next) => {
  console.log('Request:', req.method, req.url);
  next(); // Keyingi middleware'ga o'tish
});
```

## 6. **require() va import o'rtasidagi farq?**

**require() (CommonJS):**
- Node.js'da default
- Sinxron yuklash
- Dinamik import qilish mumkin

**import (ES6 Modules):**
- Zamonaviy standart
- Statik import
- Tree-shaking imkoniyati

## 7. **process.nextTick() va setImmediate() farqi?**

- **process.nextTick()**: Joriy operatsiya tugaganidan keyin, event loop'ning keyingi fazasidan oldin ishlaydi
- **setImmediate()**: Check fazasida, I/O operatsiyalaridan keyin ishlaydi

## 8. **Streams nima va qanday turlari bor?**

Streams - katta hajmdagi ma'lumotlarni qism-qism qayta ishlash imkonini beradi.

**Turlari:**
- **Readable**: Ma'lumot o'qish
- **Writable**: Ma'lumot yozish
- **Duplex**: O'qish va yozish
- **Transform**: Ma'lumotni o'zgartirish

```javascript
const readStream = fs.createReadStream('input.txt');
const writeStream = fs.createWriteStream('output.txt');
readStream.pipe(writeStream);
```

## 9. **Error handling qanday amalga oshiriladi?**

```javascript
// Try-catch
try {
  const data = JSON.parse(jsonString);
} catch (error) {
  console.error('Xatolik:', error.message);
}

// Promise
promise
  .catch(err => console.error(err));

// Event emitters
server.on('error', (err) => {
  console.error('Server xatolik:', err);
});
```

## 10. **Clustering nima va nima uchun kerak?**

Clustering - bir nechta Node.js jarayonlarini ishga tushirish orqali CPU yadrolari soniga qarab masshtablash imkonini beradi:

```javascript
const cluster = require('cluster');
const os = require('os');

if (cluster.isMaster) {
  const cpuCount = os.cpus().length;
  for (let i = 0; i < cpuCount; i++) {
    cluster.fork();
  }
} else {
  // Worker jarayoni
  require('./app');
}
```

## 11. **Buffer nima?**

Buffer - binary ma'lumotlarni to'g'ridan-to'g'ri xotirada saqlash va qayta ishlash uchun ishlatiladi:

```javascript
const buf = Buffer.from('Salom');
console.log(buf); // <Buffer 53 61 6c 6f 6d>
```

## 12. **package.json va package-lock.json farqi?**

- **package.json**: Loyihaning meta-ma'lumotlari va dependencies versiyalari
- **package-lock.json**: Aniq versiyalar va dependency tree'ning to'liq tasviri

Ushbu savollar Node.js intervyularida eng ko'p uchraydigan mavzularni qamrab oladi. Amaliy tajriba va kodlash mashqlari bilan birga bu bilimlar intervyuda muvaffaqiyat qozonishga yordam beradi.

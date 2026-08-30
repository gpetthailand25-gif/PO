# Bakery Purchasing / MRP / BOM — Deploy Guide

มี 2 ทางเลือกในการ Deploy **Frontend** (`bakery-frontend/`) — เลือกทางเดียวพอ:

- **Path A: GitHub Pages** — ถ้า repo `PO` ที่เปิด GitHub Pages ไว้อยู่แล้ว (เจอ 404 ตอนนี้)
  ทำตาม Path A เพื่อแก้ปัญหาปัจจุบันโดยตรง ไม่ต้องมี Firebase Secret เพิ่ม
- **Path B: Firebase Hosting** — ถ้าอยากใช้ Project `po-innova` ที่ให้ Config มา

> **สำคัญ (ทั้งสองทาง):** ทั้ง GitHub Pages และ Firebase Hosting เสิร์ฟได้แค่ไฟล์ Static
> เท่านั้น — **ไม่รองรับ Backend Node.js** (`bakery-mrp/`) ที่ทำไว้ ต้อง Deploy Backend
> แยกไปที่อื่น (ดูท้ายเอกสาร) แล้วตั้ง Secret ให้ Frontend รู้จัก ไม่งั้นเว็บจะขึ้นแต่
> Login/โหลดข้อมูลไม่ได้เลย

---

## Path A: GitHub Pages (แก้ 404 ที่เจอตอนนี้)

สาเหตุของ 404 คือ Push โค้ด Source ดิบขึ้นไปตรง ๆ (ยังไม่ผ่าน `npm run build`) แล้วเปิด
Pages เสิร์ฟตรงนั้นเลย ซึ่งไม่มี `index.html` ที่ใช้งานได้จริงอยู่ที่ตำแหน่งรากของ repo

### 1. ตั้งค่า Pages ให้ Build ผ่าน GitHub Actions แทน

Repo `PO` บน GitHub → **Settings → Pages** → หัวข้อ **Build and deployment** →
เปลี่ยน **Source** จาก "Deploy from a branch" เป็น **"GitHub Actions"**

### 2. (ถ้ามี Backend Deploy แล้ว) ตั้ง Secret ให้ Frontend รู้จัก

**Settings → Secrets and variables → Actions → New repository secret**
- ชื่อ: `VITE_API_BASE_URL`
- ค่า: URL เต็มของ Backend เช่น `https://bakery-mrp-api.onrender.com`

ถ้ายังไม่มี Backend deploy ข้ามข้อนี้ไปก่อนได้ (เว็บจะขึ้น แต่ Login ไม่ได้จนกว่าจะตั้งค่านี้)

### 3. Push โค้ดชุดใหม่ทับของเดิม (มีไฟล์ `.github/workflows/gh-pages-deploy.yml` เพิ่มมาแล้ว)

```bash
# แตกไฟล์ zip ใหม่ทับโฟลเดอร์ repo เดิม (หรือ clone ใหม่แล้ว copy ทับ)
git add .
git commit -m "fix: build and deploy via GitHub Actions instead of raw source"
git push
```

ไปที่แท็บ **Actions** ของ repo จะเห็น Workflow "Deploy Frontend to GitHub Pages" กำลังรัน
รอสัก 1-2 นาที เสร็จแล้วเปิดดูที่ URL เดิม:

```
https://gpetthailand25-gif.github.io/PO/
```

ควรเห็นหน้า Login ของระบบแล้ว (ถ้ายังไม่ตั้ง `VITE_API_BASE_URL` การกด Login จะ error
เพราะยังไม่เจอ Backend — เป็นเรื่องคาดหมายได้ ไม่ใช่ตั้งผิด)

> หมายเหตุ: ไฟล์ `vite.config.js` และ `src/main.jsx` ถูกแก้ให้รองรับการรันใต้ Subpath
> `/PO/` โดยอัตโนมัติแล้ว (Workflow คำนวณจากชื่อ repo ให้เอง ไม่ต้องแก้ค่าเอง)

---

## Path B: Firebase Hosting (Project `po-innova`)

### 1. สร้าง Firebase Service Account Secret

```bash
npm install -g firebase-tools
firebase login
cd bakery-frontend
firebase init hosting:github
```

จะสร้าง Secret ชื่อ `FIREBASE_SERVICE_ACCOUNT_PO_INNOVA` ให้อัตโนมัติ — ถ้า CLI ถามจะ
สร้าง Workflow ไฟล์ให้เองด้วย ให้ตอบไม่ (มีให้แล้วใน `.github/workflows/firebase-hosting-*.yml`)

ทำมือแทนได้ถ้าไม่อยากใช้คำสั่งนี้:
1. Firebase Console → Project Settings → Service Accounts → Generate New Private Key
2. GitHub repo → Settings → Secrets → New repository secret →
   ชื่อ `FIREBASE_SERVICE_ACCOUNT_PO_INNOVA` → ค่า = เนื้อหาไฟล์ JSON ทั้งไฟล์

### 2. ตั้ง Secret `VITE_API_BASE_URL` (เหมือน Path A ข้อ 2)

### 3. Push ขึ้น GitHub

Workflow `.github/workflows/firebase-hosting-merge.yml` จะ Build + Deploy อัตโนมัติ
เปิดดูได้ที่ `https://po-innova.web.app`

---

## Deploy Backend (`bakery-mrp/`) — ต้องทำแยก ไม่ว่าจะเลือก Path ไหน

Backend เป็น Node.js server ธรรมดา (`npm start`, ฟัง `process.env.PORT` อยู่แล้ว)
Deploy ได้กับผู้ให้บริการที่รองรับ Node ตรง ๆ:

- **Render.com** — ง่ายสุด เชื่อม GitHub repo แล้วเลือก Root Directory เป็น `bakery-mrp`,
  Build Command `npm install`, Start Command `npm start`
- **Railway.app** — คล้ายกัน
- **Google Cloud Run** — เข้ากับ Firebase Project เดียวกัน (`po-innova`) ได้ลงตัวที่สุด
  แต่ต้องมี `Dockerfile` (ยังไม่ได้เตรียมไว้ในโปรเจกต์นี้)

ได้ URL แล้วเอาไปใส่ Secret `VITE_API_BASE_URL` ตามขั้นตอนข้างบน แล้ว Push ใหม่อีกครั้ง
(หรือกด "Re-run all jobs" ที่แท็บ Actions) เพื่อให้ Frontend Build ใหม่ด้วยค่านี้

## ⚠️ สิ่งที่ยังต้องทำเอง (ยืนยันไม่ได้จากฝั่งนี้ เพราะ Sandbox ไม่มีเน็ต)

- ยังไม่เคยรัน `npm install` จริงในทั้งสองโปรเจกต์ (ไม่มี `package-lock.json` มาด้วย) —
  รันครั้งแรกที่เครื่องคุณแล้ว commit `package-lock.json` เข้า Git ด้วย (Workflow ใช้ `npm ci`)
- ยังไม่เคย Deploy จริงกับ repo/Firebase Project ของคุณ — ถ้า Actions log ขึ้น error
  ส่งภาพหรือข้อความ log กลับมาได้เลย

## Firebase Config ที่ให้มา (สำหรับ Path B)

```js
const firebaseConfig = {
  apiKey: "AIzaSyBxEa_yKLTzGz-ICV_bsZDVU1NHx8MiOTg",
  authDomain: "po-innova.firebaseapp.com",
  projectId: "po-innova",
  storageBucket: "po-innova.firebasestorage.app",
  messagingSenderId: "847160966036",
  appId: "1:847160966036:web:1688cf337d184c6c9bed81",
  measurementId: "G-TMKYJCCEYC"
};
```

ยังไม่ได้ฝังเข้าโค้ด Frontend เพราะใช้ Firebase แค่ Hosting เท่านั้น (Auth ยังเป็น JWT
ที่ทำเอง) — ถ้าอยากเปลี่ยนไปใช้ Firebase Authentication/Firestore จริง บอกได้เลย
เป็นงานแยกที่ต้องปรับโครงสร้างเพิ่ม

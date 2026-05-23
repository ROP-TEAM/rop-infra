# OSRM Service Setup & Deployment Guide

เมื่อเราวิ่งเร็วขึ้นเราจะมองภาพรอบข้างไม่ค่อยชัด

## 🏗️ แผนภาพสถาปัตยกรรม (Workflow)
required project structure 
osrm/
├── data/
│   ├── thailand-latest.osm.pbf
│   ├── thailand-latest.osrm
│   ├── thailand-latest.osrm.cells
│   └── (ไฟล์อื่นๆ ที่ได้จากการ pre-process)
└── Dockerfile

0. download osrm backend engine from https://github.com/Project-OSRM/osrm-backend package 

1. **Download:** โหลดไฟล์ `.osm.pbf` จาก Geofabrik
    - than change the suflix from version to latest (eg. from thailand-1992 to thailand-lastest)

2. **Preprocess:** ใช้คลัง `osrm/osrm-backend` มารัน Extract และ Contract/Customize บนเครื่องเพื่อแปลงเป็นฟอร์แมต OSRM
    - follow the instruction from https://github.com/Project-OSRM/osrm-backend .md file 

3. **Build Image:** เขียน `Dockerfile` เพื่อคัดลอก Data ที่เสร็จแล้วเข้าไปรวมกับเอนจินรันระบบ
    ```
    <!-- (choose the same Engine in the pre-process step) -->
    FROM ghcr.io/project-osrm/osrm-backend:v26.5.0-amd64-alpine 

    RUN mkdir -p /data

    COPY ./data/ /data/

    EXPOSE 5000

    <!-- (the config of the service in this line, you can read the document from osrm project to config it. eg. --max-table-size to 444 id) -->
    ENTRYPOINT ["osrm-routed", "--algorithm", "mld", "--max-table-size", "444", "/data/thailand-latest.osrm"] 
    ```

4. **Publish:** ส่งขึ้น `ghcr.io` เพื่อนำไป Deploy ต่อได้ทันที
    docker build -t ghcr.io/rop-team/osrm-thailand:latest .

    docker login ghcr.io -u YOUR_GITHUB_USERNAME --password-stdin

    docker push ghcr.io/ORGANIZATION_NAME/osrm-thailand:latest

5. waiting for build and employ
    if the service in a server did not upload, contact ใครสักคนนที่ดูserverอยู่

---

## 🛠️ ขั้นตอนที่ 1: เตรียมข้อมูลและรัน Preprocess บนเครื่อง (Local Machine)

สร้างโฟลเดอร์สำหรับทำงานและดาวน์โหลดข้อมูลแผนที่ก่อน (ตัวอย่างนี้จะใช้แผนที่ประเทศไทย)

```bash
# 1. สร้างโฟลเดอร์และเข้าไปข้างใน
mkdir osrm-service && cd osrm-service
mkdir data

# 2. ดาวน์โหลดไฟล์แผนที่ประเทศไทย (.pbf) จาก Geofabrik
curl -o data/thailand-latest.osm.pbf [https://download.geofabrik.de/asia/thailand-latest.osm.pbf](https://download.geofabrik.de/asia/thailand-latest.osm.pbf)
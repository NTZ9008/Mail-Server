# 📧 Ultimate Home Lab Mail Server (Hybrid Cloud-Premise)

![Project Status](https://img.shields.io/badge/Status-Operational-success)
![Docker](https://img.shields.io/badge/Docker-Containerized-blue)
![Security](https://img.shields.io/badge/SSL-Let's_Encrypt-green)

> **"Turn a residential Mini PC into a production-grade Mail Server, bypassing ISP restrictions (CGNAT & Port 25 block)."**

---

## 📖 Overview (ภาพรวมโปรเจกต์)
โปรเจกต์นี้คือการสร้าง Mail Server ส่วนตัวโดยใช้ฮาร์ดแวร์ที่บ้าน (Mini PC) แม้ว่าจะติดข้อจำกัดเรื่อง Network พื้นฐานของเน็ตบ้าน (Dynamic IP, NAT) และการบล็อก Port 25 โดยใช้เทคนิค **Tunneling** ผ่าน VPS และใช้ **SMTP Relay** เพื่อให้สามารถรับ-ส่งอีเมลเข้า Inbox ผู้รับได้จริง 100%

## 🏗 Network Topology (โครงสร้างระบบ)
การออกแบบระบบเป็นแบบ **Hybrid Topology** เชื่อมต่อระหว่าง Public Cloud และ Local Server ผ่าน VPN Tunnel

```mermaid
graph TD
    Client["📧 External User"] --> CF["🛡️ Cloudflare DNS"]
    CF --> VPS["☁️ Oracle VPS (Public Gateway)"]
    VPS -- "Tailscale Tunnel" --> MiniPC["🏠 Home Mini PC"]
    MiniPC -- "SMTP Relay (Port 587)" --> Brevo["🚀 Brevo"]
    Brevo --> Destination["📬 Recipient Inbox"]

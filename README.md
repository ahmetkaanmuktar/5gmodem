import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
from sklearn.cluster import KMeans
from sklearn.preprocessing import StandardScaler
import time
import random
from datetime import datetime, timedelta

class SolarNanoCellModem:
    def __init__(self, solar_efficiency=0.23, battery_capacity=10000, initial_battery=5000):
        # Sistem parametreleri
        self.solar_efficiency = solar_efficiency
        self.battery_capacity = battery_capacity
        self.battery_level = initial_battery

        # Ağ parametreleri
        self.connected_devices = 0
        self.max_devices = 50
        self.data_usage = []
        self.signal_quality = 0.85

        # Güvenlik parametreleri
        self.threat_database = {
            "ddos": {"pattern": "yüksek trafik artışı", "risk": "yüksek"},
            "mitm": {"pattern": "anormal rota değişikliği", "risk": "kritik"},
            "bruteforce": {"pattern": "çoklu başarısız giriş", "risk": "orta"}
        }
        self.detected_threats = []

        # Performans metrikleri
        self.uptime = 0
        self.energy_consumption_history = []
        self.network_optimization_stats = {
            "optimizations_performed": 0,
            "bandwidth_saved": 0,
            "latency_improvement": 0
        }

        # Beamforming parametreleri
        self.beamforming_active = True
        self.beamforming_angles = []

        print("5G Solar NanoCell Modem simülasyonu başlatıldı.")

    def solar_charging(self, sun_intensity=0.7, duration_hours=1):
        generated_power = sun_intensity * self.solar_efficiency * 1000 * duration_hours
        charge_amount = min(generated_power, self.battery_capacity - self.battery_level)
        self.battery_level += charge_amount

        print(f"Güneş enerjisiyle {charge_amount:.2f} mAh şarj eklendi. Batarya: {self.battery_level:.2f}/{self.battery_capacity} mAh")
        return charge_amount

    def power_consumption(self, active_connections, data_transfer_rate):
        total = 50 + active_connections * 2 + data_transfer_rate * 0.5
        if self.beamforming_active:
            total += 20
        self.energy_consumption_history.append(total)
        return total

    def network_usage(self, duration_minutes=60):
        print(f"\n{duration_minutes} dakikalık ağ kullanımı simülasyonu başlatılıyor...")
        time_points = duration_minutes
        connection_profile = np.clip(np.random.normal(25, 10, time_points), 0, self.max_devices)

        total_data, total_power = 0, 0
        data_usage_profile = []

        for t in range(time_points):
            active = int(connection_profile[t])
            rate = active * np.random.uniform(0.5, 2.0)

            data_usage_profile.append(rate)
            total_data += rate

            consumed = self.power_consumption(active, rate)
            total_power += consumed
            self.battery_level -= consumed / 60

            if self.battery_level < self.battery_capacity * 0.2:
                print(f"UYARI: Batarya seviyesi kritik! ({self.battery_level:.2f}/{self.battery_capacity} mAh)")

            if t % 10 == 0:
                print(f"Dakika {t}: {active} aktif cihaz, {rate:.2f} Mbps veri hızı")

        self.data_usage.extend(data_usage_profile)

        print(f"\nAğ simülasyonu tamamlandı.\nOrtalama veri hızı: {np.mean(data_usage_profile):.2f} Mbps\nToplam veri transferi: {total_data:.2f} MB\nToplam güç tüketimi: {total_power:.2f} mAh\nKalan batarya: {self.battery_level:.2f}/{self.battery_capacity} mAh")

        return {
            "duration_minutes": duration_minutes,
            "average_active_devices": np.mean(connection_profile),
            "average_data_rate_mbps": np.mean(data_usage_profile),
            "peak_data_rate_mbps": np.max(data_usage_profile),
            "total_data_transferred_mb": total_data,
            "total_power_consumed_mah": total_power,
            "remaining_battery_mah": self.battery_level
        }

    def optimize_network(self):
        print("\nAğ optimizasyonu başlatılıyor...")
        if len(self.data_usage) < 10:
            print("Yeterli veri yok. Optimizasyon için daha fazla ağ kullanımı gerekiyor.")
            return {"status": "yetersiz veri"}

        scaled = StandardScaler().fit_transform(np.array(self.data_usage).reshape(-1, 1))
        kmeans = KMeans(n_clusters=3, random_state=42).fit(scaled)
        centers = StandardScaler().inverse_transform(kmeans.cluster_centers_)

        low, high = min(centers)[0], max(centers)[0]
        allocation = {}

        for i, c in enumerate(kmeans.labels_[-min(len(kmeans.labels_), self.max_devices):]):
            val = centers[c][0]
            if val < low * 1.2:
                allocation[f"device_{i}"] = "düşük öncelik"
            elif val > high * 0.8:
                allocation[f"device_{i}"] = "yüksek öncelik"
            else:
                allocation[f"device_{i}"] = "normal öncelik"

        if self.beamforming_active and allocation:
            angles = np.linspace(0, 350, len(allocation))
            np.random.shuffle(angles)
            self.beamforming_angles = angles.tolist()
            print(f"Beamforming optimizasyonu: {len(allocation)} cihaz için açılar hesaplandı.")

        saved = np.mean(self.data_usage) * np.random.uniform(0.1, 0.25)
        latency = np.random.uniform(5, 15)

        self.network_optimization_stats["optimizations_performed"] += 1
        self.network_optimization_stats["bandwidth_saved"] += saved
        self.network_optimization_stats["latency_improvement"] += latency

        print(f"Ağ optimizasyonu tamamlandı:\nBant genişliği tasarrufu: {saved:.2f} Mbps\nGecikme iyileştirmesi: {latency:.2f} ms")
        return {"status": "başarılı", "bandwidth_saved_mbps": saved, "latency_improvement_ms": latency, "beamforming_optimized": self.beamforming_active}

    def security_scan(self):
        print("\nGüvenlik taraması başlatılıyor...")
        threats = []

        if np.random.random() < 0.3:
            threat_type = random.choice(list(self.threat_database))
            t = self.threat_database[threat_type]
            detected = {
                "type": threat_type,
                "pattern": t["pattern"],
                "risk_level": t["risk"],
                "source_ip": f"192.168.1.{random.randint(2, 254)}",
                "detection_time": datetime.now().strftime("%Y-%m-%d %H:%M:%S"),
                "status": "engellendi"
            }
            self.detected_threats.append(detected)
            threats.append(detected)
            print(f"TEHDİT TESPİT EDİLDİ: {threat_type} - {t['risk']}")

        print(f"Güvenlik taraması tamamlandı - Tespit edilen tehdit: {len(threats)}")
        return threats

    def system_report(self):
        self.uptime += 1
        recent_threats = [t for t in self.detected_threats if (datetime.now() - datetime.strptime(t["detection_time"], "%Y-%m-%d %H:%M:%S")) < timedelta(hours=24)]
        trend = "artıyor" if len(self.data_usage) > 10 and np.mean(self.data_usage[-5:]) > np.mean(self.data_usage[-10:-5]) else "sabit"

        report = {
            "timestamp": datetime.now().strftime("%Y-%m-%d %H:%M:%S"),
            "uptime_hours": self.uptime,
            "battery_status": {
                "level": self.battery_level,
                "capacity": self.battery_capacity,
                "percentage": (self.battery_level / self.battery_capacity) * 100
            },
            "network_stats": {
                "connected_devices": self.connected_devices,
                "signal_quality": self.signal_quality * 100,
                "data_usage_trend": trend
            },
            "security_status": {
                "threats_detected_total": len(self.detected_threats),
                "recent_threats": len(recent_threats)
            },
            "optimization_stats": self.network_optimization_stats,
            "recommendations": []
        }

        if report["battery_status"]["percentage"] < 30:
            report["recommendations"].append("Batarya seviyesi düşük. Güneş panelinin konumunu optimize edin.")
        if self.network_optimization_stats["optimizations_performed"] == 0:
            report["recommendations"].append("Ağ optimizasyonu henüz çalıştırılmadı. Performansı artırmak için optimizasyon yapın.")
        if trend == "artıyor":
            report["recommendations"].append("Veri kullanımında artış eğilimi var. Trafik analizi yapılması önerilir.")

        return report

    def visualize_data(self):
        if not self.energy_consumption_history or not self.data_usage:
            print("Görselleştirme için yeterli veri yok.")
            return

        fig, (ax1, ax2) = plt.subplots(2, 1, figsize=(10, 8))

        ax1.plot(self.energy_consumption_history, 'r-', label='Enerji Tüketimi (mAh)')
        ax1.set_title('Enerji Tüketimi')
        ax1.set_xlabel('Zaman (dakika)')
        ax1.set_ylabel('Tüketim (mAh)')
        ax1.grid(True)
        ax1.legend()

        ax2.plot(self.data_usage, 'b-', label='Veri Kullanımı (Mbps)')
        ax2.set_title('Veri Kullanımı')
        ax2.set_xlabel('Zaman (dakika)')
        ax2.set_ylabel('Mbps')
        ax2.grid(True)
        ax2.legend()

        plt.tight_layout()
        plt.show()


def run_simulation():
    modem = SolarNanoCellModem(solar_efficiency=0.23, battery_capacity=10000, initial_battery=7500)

    print("\n[09:00] Başlangıç")
    modem.solar_charging(0.6, 1)
    modem.network_usage(60)

    print("\n[12:00] Yüksek Kullanım")
    modem.solar_charging(0.9, 1)
    modem.network_usage(60)

    print("\n[13:00] Ağ Optimizasyonu")
    modem.optimize_network()

    print("\n[13:30] Güvenlik Taraması")
    modem.security_scan()

    print("\n[18:00] Düşük Kullanım")
    modem.solar_charging(0.3, 1)
    modem.network_usage(60)

    print("\n[19:00] Günlük Rapor")
    report = modem.system_report()

    print("\nGÜNLÜK SİSTEM RAPORU")
    print("="*50)
    print(f"Tarih: {report['timestamp']}")
    print(f"Çalışma Süresi: {report['uptime_hours']} saat")
    print(f"Batarya: %{report['battery_status']['percentage']:.1f} ({report['battery_status']['level']:.0f}/{report['battery_status']['capacity']})")
    print(f"Sinyal Kalitesi: %{report['network_stats']['signal_quality']:.1f}")
    print(f"Veri Kullanım Eğilimi: {report['network_stats']['data_usage_trend']}")
    print(f"Toplam Tehdit: {report['security_status']['threats_detected_total']}")
    print("\nÖneriler:")
    for rec in report["recommendations"]:
        print(f"- {rec}")

    print("\n[19:30] Görselleştirme")
    modem.visualize_data()

    print("\nSİMÜLASYON TAMAMLANDI")


if __name__ == "__main__":
    run_simulation()

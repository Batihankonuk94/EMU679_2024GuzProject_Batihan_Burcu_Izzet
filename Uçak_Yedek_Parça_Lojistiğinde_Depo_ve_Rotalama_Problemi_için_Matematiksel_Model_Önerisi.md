# EMU679_2024GuzProject_Batihan_Burcu_Izzet
EMU679_2024GuzProject_Batihan_Burcu_Izzet
from gurobipy import Model, GRB, quicksum

# Model oluşturma
model = Model("Bakım_Merkezleri_Optimizasyonu")

# Setler
I = range(1, 11)  # Yedek parça seti
A = ['sivil_arac 1', 'sivil_arac 2', 'sivil_arac 3', 'helikopter 1', 'helikopter 2', 'helikopter 3']
B = ['Çiğli', 'Eskişehir', 'Konya', 'Mürted', 'Merzifon', 'Bandırma', 'Malatya', 'Diyarbakır', 'Balıkesir']
T = range(1, 6)

# Parametreler
D = {(3, 'Malatya', 1): 1, (10, 'Mürted', 1): 1, (10, 'Eskişehir', 1): 1, (1, 'Konya', 1): 1, (9, 'Bandırma', 2): 1,
     (10, 'Eskişehir', 3): 1,
     (4, 'Konya', 3): 1, (3, 'Çiğli', 3): 1, (3, 'Eskişehir', 3): 1, (3, 'Balıkesir', 3): 1, (6, 'Mürted', 4): 1,
     (8, 'Diyarbakır', 4): 1,
     (10, 'Merzifon', 4): 1, (7, 'Çiğli', 4): 1, (3, 'Bandırma', 4): 1, (7, 'Eskişehir', 5): 1, (8, 'Bandırma', 5): 1,
     (1, 'Bandırma', 5): 1,
     (8, 'Malatya', 5): 1, (7, 'Konya', 5): 1, (8, 'Diyarbakır', 5): 1, (6, 'Merzifon', 5): 1}

S = {1:50,2:1,3:6,4:5,5:1,6:17,7:2,8:1,9:200,10:6}  # Depo stokları
K = {'sivil_arac 1': 500, 'sivil_arac 2': 500, 'sivil_arac 3': 500, 'helikopter 1': 1500, 'helikopter 2': 1500,
     'helikopter 3': 1500}  # Araç kapasitesi
P = {1: 2, 2: 3, 3: 9, 4: 13, 5: 790, 6: 1, 7: 1.3, 8: 14, 9: 0.3, 10: 2.5}  # Parça ağırlıkları
M = {1: 15, 2: 15, 3: 25, 4: 21, 5: 60, 6: 10, 7: 28, 8: 25, 9: 28, 10: 28}  # Stok maliyeti
O = {1: 10, 2: 12.6, 3: 15, 4: 16.8, 5: 100, 6: 5.1, 7: 33.6, 8: 29.4, 9: 33.6, 10: 33.6}  # Acil durum stok maliyeti
G = {'Çiğli': 608, 'Eskişehir': 243, 'Konya': 289, 'Mürted': 2, 'Merzifon': 332, 'Bandırma': 510, 'Malatya': 696,
     'Diyarbakır': 933, 'Balıkesir': 539}
V = {'sivil_arac 1': 1, 'sivil_arac 2': 1, 'sivil_arac 3': 1, 'helikopter 1': 45, 'helikopter 2': 45,
     'helikopter 3': 45}
F = {('Çiğli', 'sivil_arac 1'): 6.75, ('Eskişehir', 'sivil_arac 1'): 2.7, ('Konya', 'sivil_arac 1'): 3.21,
     ('Mürted', 'sivil_arac 1'): 0.02, ('Merzifon', 'sivil_arac 1'): 3.69, ('Bandırma', 'sivil_arac 1'): 5.67,
     ('Malatya', 'sivil_arac 1'): 7.73, ('Diyarbakır', 'sivil_arac 1'): 10.37, ('Balıkesir', 'sivil_arac 1'): 6,
     ('Çiğli', 'helikopter 1'): 2.5, ('Eskişehir', 'helikopter 1'): 0.98, ('Konya', 'helikopter 1'): 1.2,
     ('Mürted', 'helikopter 1'): 0.008, ('Merzifon', 'helikopter 1'): 1.4, ('Bandırma', 'helikopter 1'): 2,
     ('Malatya', 'helikopter 1'): 2.8, ('Diyarbakır', 'helikopter 1'): 3.7, ('Balıkesir', 'helikopter 1'): 2.16}

# F sözlüğünü diğer araçlar için genişlet
for b in B:
    for a in ['sivil_arac 2', 'sivil_arac 3']:
        F[(b, a)] = F[(b, 'sivil_arac 1')]
    for a in ['helikopter 2', 'helikopter 3']:
        F[(b, a)] = F[(b, 'helikopter 1')]

R = {'Çiğli':0.1,'Eskişehir':0.05,'Konya':0.1,'Mürted':0.1,'Merzifon':0.2,'Bandırma':0.2,'Malatya':0.1,'Diyarbakır':0.05,'Balıkesir':0.1}  # Öncelik seviyesi
  # Öncelik seviyesi
beta = 20  # Öncelik seviyesi etkisi

# Talep noktalarını ve günlerini belirle
demand_tuples = [(i, b, t) for (i, b, t) in D.keys()]
active_locations_by_day = {t: set(b for (_, b, t2) in demand_tuples if t2 == t) for t in T}

# Değişkenler
x = {}  # Teslimat miktarları
for i, b, t in demand_tuples:
    for a in A:
        x[i, a, b, t] = model.addVar(vtype=GRB.CONTINUOUS, lb=0, name=f"x_{i}_{a}_{b}_{t}")

z = {}  # Araç atama
for t in T:
    for a in A:
        for b in active_locations_by_day[t]:
            z[a, b, t] = model.addVar(vtype=GRB.BINARY, name=f"z_{a}_{b}_{t}")

w = {}  # Rota seçimi
for t in T:
    for a in A:
        for i in ['Fabrika'] + list(active_locations_by_day[t]):
            for j in ['Fabrika'] + list(active_locations_by_day[t]):
                if i != j:
                    w[a, i, j, t] = model.addVar(vtype=GRB.BINARY, name=f"w_{a}_{i}_{j}_{t}")

y = model.addVars(I, T, vtype=GRB.CONTINUOUS, lb=0, name="y")  # Acil stok

# Amaç fonksiyonu
obj = (
        quicksum(M[i] * x[i, a, b, t] for (i, a, b, t) in x.keys()) +
        quicksum(O[i] * y[i, t] for i in I for t in T) +
        quicksum(G[b] * V[a] * F[(b, a)] * x[i, a, b, t] for (i, a, b, t) in x.keys() if (b, a) in F)
)
model.setObjective(obj, GRB.MINIMIZE)

# Kısıtlar
# 1. Talep karşılama kısıtı
for i, b, t in demand_tuples:
    c2=model.addConstr(
        quicksum(x[i, a, b, t] for a in A) == D[i, b, t],
        name=f"demand_{i}_{b}_{t}"
    )

# 2. Araç-teslimat bağlantı kısıtı
for t in T:
    for b in active_locations_by_day[t]:
        for a in A:
            relevant_parts = [i for (i, b2, t2) in demand_tuples if b2 == b and t2 == t]
            if relevant_parts:
                c3=model.addConstr(
                    quicksum(x[i, a, b, t] for i in relevant_parts) <= sum(D[i, b, t] for i in relevant_parts) * z[
                        a, b, t],
                    name=f"delivery_link_{a}_{b}_{t}"
                )

# 3. Araç kapasite kısıtı
for t in T:
    for a in A:
        c4=model.addConstr(
            quicksum(P[i] * x[i, a, b, t] for (i, a2, b, t2) in x.keys() if a2 == a and t2 == t) <= K[a],
            name=f"capacity_{a}_{t}"
        )

# 4. Rota bağlantı kısıtları
for t in T:
    for a in A:
        # Her araç fabrikadan çıkmalı
        c5=model.addConstr(
            quicksum(w[a, 'Fabrika', j, t] for j in active_locations_by_day[t]) <= 1,
            name=f"depart_depot_{a}_{t}"
        )

        # Her lokasyona giren ve çıkan araç sayısı eşit olmalı
        for b in active_locations_by_day[t]:
            c6=model.addConstr(
                quicksum(w[a, i, b, t] for i in ['Fabrika'] + list(active_locations_by_day[t]) if i != b) ==
                quicksum(w[a, b, j, t] for j in ['Fabrika'] + list(active_locations_by_day[t]) if j != b),
                name=f"flow_{a}_{b}_{t}"
            )

            # Eğer bir lokasyona teslimat varsa, o lokasyona bir araç gitmeli
            c7=model.addConstr(
                quicksum(w[a, i, b, t] for i in ['Fabrika'] + list(active_locations_by_day[t]) if i != b) >= z[a, b, t],
                name=f"visit_if_delivery_{a}_{b}_{t}"
            )



# 5. Depo stok yenilemesi ve acil stok kısıtı
M_big = 100000  # Büyük bir sayı

# Yardımcı binary değişken
v = {}
for i in I:
    for t in T:
        v[i, t] = model.addVar(vtype=GRB.BINARY, name=f"v_{i}_{t}")

for i in I:
    for t in T:
        # Her gün için toplam teslimat miktarını hesapla
        total_delivery = quicksum(x[i, a, b, t] for (i2, a, b, t2) in x.keys() if i2 == i and t2 == t)

        # Binary değişken için kısıt: v[i,t] = 1 if total_delivery > S[i]
        c8=model.addConstr(
            total_delivery - S[i] <= M_big * v[i, t],
            name=f"emergency_trigger1_{i}_{t}"
        )
        model.addConstr(
            total_delivery - S[i] + 0.0001 >= (-M_big) * (1 - v[i, t]),
            name=f"emergency_trigger2_{i}_{t}"
        )

        # Eğer teslimat miktarı stoktan fazlaysa, aradaki fark kadar acil stok üretilmeli
        c9=model.addConstr(
            y[i, t] >= total_delivery - S[i],
            name=f"emergency_stock_requirement_{i}_{t}"
        )

        # Acil stok sadece gerektiğinde üretilmeli (stok yetersiz olduğunda)
        c10=model.addConstr(
            y[i, t] <= M_big * v[i, t],
            name=f"emergency_stock_control_{i}_{t}"
        )

# 6. Öncelik seviyesine bağlı teslim süresi kısıtı
for t in T:
    for b in B:
        c11=model.addConstr(
            quicksum(z[a, b, t] * F[(b, a)] for a in A if (b, a) in F and b in active_locations_by_day[t]) <= beta / R[
                b],
            name=f"priority_delivery_time_{b}_{t}"
        )

# 6. Alt tur eliminasyon kısıtları
u = {}
for t in T:
    for a in A:
        for b in active_locations_by_day[t]:
            u[a, b, t] = model.addVar(lb=0, ub=len(B), vtype=GRB.CONTINUOUS, name=f"u_{a}_{b}_{t}")

for t in T:
    for a in A:
        for i in active_locations_by_day[t]:
            for j in active_locations_by_day[t]:
                if i != j and (a, i, j, t) in w:
                    c12=model.addConstr(
                        u[a, i, t] - u[a, j, t] + len(B) * w[a, i, j, t] <= len(B) - 1,
                        name=f"subtour_{a}_{i}_{j}_{t}"
                    )

# Modeli çöz
model.optimize()


# Sonuçları yazdır
def print_decision_variables(x, y, z, w):
    print("\n=== KARAR DEĞİŞKENLERİNİN DEĞERLERİ ===")

    # X değişkeni (Teslimat miktarları)
    print("\n1. Teslimat Miktarları (x[i,a,b,t]):")
    print("-" * 50)
    for (i, a, b, t) in sorted(x.keys()):
        if x[i, a, b, t].X > 0.001:  # Sadece pozitif değerleri göster
            print(f"x[Parça {i}, {a}, {b}, Gün {t}] = {x[i, a, b, t].X:.2f}")

    # Y değişkeni (Acil stok miktarları)
    print("\n2. Acil Stok Miktarları (y[i,t]):")
    print("-" * 50)
    for i in I:
        for t in T:
            if y[i, t].X > 0.001:  # Sadece pozitif değerleri göster
                print(f"y[Parça {i}, Gün {t}] = {y[i, t].X:.2f}")

    # Z değişkeni (Araç atamaları)
    print("\n3. Araç Atamaları (z[a,b,t]):")
    print("-" * 50)
    for (a, b, t) in sorted(z.keys()):
        if z[a, b, t].X > 0.5:  # Binary değişken için 0.5'ten büyük değerleri göster
            print(f"z[{a}, {b}, Gün {t}] = 1")

    # W değişkeni (Rota seçimleri)
    print("\n4. Rota Seçimleri (w[a,i,j,t]):")
    print("-" * 50)
    for (a, i, j, t) in sorted(w.keys()):
        if w[a, i, j, t].X > 0.5:  # Binary değişken için 0.5'ten büyük değerleri göster
            print(f"w[{a}, {i}->{j}, Gün {t}] = 1")

    print("\n=== ÖNEMLİ İSTATİSTİKLER ===")

    # Toplam teslimat miktarı
    total_delivery = sum(x[i, a, b, t].X for (i, a, b, t) in x.keys())
    print(f"\nToplam teslimat miktarı: {total_delivery:.2f}")

    # Toplam acil stok miktarı
    total_emergency = sum(y[i, t].X for i in I for t in T)
    print(f"Toplam acil stok miktarı: {total_emergency:.2f}")

    # Toplam araç atama sayısı
    total_assignments = sum(1 for (a, b, t) in z.keys() if z[a, b, t].X > 0.5)
    print(f"Toplam araç atama sayısı: {total_assignments}")

    # Toplam rota sayısı
    total_routes = sum(1 for (a, i, j, t) in w.keys() if w[a, i, j, t].X > 0.5)
    print(f"Toplam rota sayısı: {total_routes}")


def print_decision_variables(x, y, z, w, file):
    file.write("\n=== KARAR DEĞİŞKENLERİNİN DEĞERLERİ ===\n")

    # X değişkeni (Teslimat miktarları)
    file.write("\n1. Teslimat Miktarları (x[i,a,b,t]):\n")
    file.write("-" * 50 + "\n")
    for (i, a, b, t) in sorted(x.keys()):
        if x[i, a, b, t].X > 0.001:  # Sadece pozitif değerleri göster
            file.write(f"x[Parça {i}, {a}, {b}, Gün {t}] = {x[i, a, b, t].X:.2f}\n")

    # Y değişkeni (Acil stok miktarları)
    file.write("\n2. Acil Stok Miktarları (y[i,t]):\n")
    file.write("-" * 50 + "\n")
    for i in I:
        for t in T:
            if y[i, t].X > 0.001:  # Sadece pozitif değerleri göster
                file.write(f"y[Parça {i}, Gün {t}] = {y[i, t].X:.2f}\n")

    # Z değişkeni (Araç atamaları)
    file.write("\n3. Araç Atamaları (z[a,b,t]):\n")
    file.write("-" * 50 + "\n")
    for (a, b, t) in sorted(z.keys()):
        if z[a, b, t].X > 0.5:  # Binary değişken için 0.5'ten büyük değerleri göster
            file.write(f"z[{a}, {b}, Gün {t}] = 1\n")

    # W değişkeni (Rota seçimleri)
    file.write("\n4. Rota Seçimleri (w[a,i,j,t]):\n")
    file.write("-" * 50 + "\n")
    for (a, i, j, t) in sorted(w.keys()):
        if w[a, i, j, t].X > 0.5:  # Binary değişken için 0.5'ten büyük değerleri göster
            file.write(f"w[{a}, {i}->{j}, Gün {t}] = 1\n")

    file.write("\n=== ÖNEMLİ İSTATİSTİKLER ===\n")

    # Toplam teslimat miktarı
    total_delivery = sum(x[i, a, b, t].X for (i, a, b, t) in x.keys())
    file.write(f"\nToplam teslimat miktarı: {total_delivery:.2f}\n")

    # Toplam acil stok miktarı
    total_emergency = sum(y[i, t].X for i in I for t in T)
    file.write(f"Toplam acil stok miktarı: {total_emergency:.2f}\n")

    # Toplam araç atama sayısı
    total_assignments = sum(1 for (a, b, t) in z.keys() if z[a, b, t].X > 0.5)
    file.write(f"Toplam araç atama sayısı: {total_assignments}\n")

    # Toplam rota sayısı
    total_routes = sum(1 for (a, i, j, t) in w.keys() if w[a, i, j, t].X > 0.5)
    file.write(f"Toplam rota sayısı: {total_routes}\n")


def write_daily_plans(route_func, x, z, w, y, I, A, B, T, D, file):
    for t in T:
        file.write(f"\nGün {t} planı:\n")
        file.write(f"Bu gün için talepler:\n")
        day_demands = [(i, b) for (i, b, t2) in demand_tuples if t2 == t]
        for i, b in day_demands:
            file.write(f"  {b}: Parça {i} - {D[i, b, t]} birim\n")

        file.write("\nRota ve teslimatlar:\n")
        for a in A:
            route = route_func(a, t)
            if route:
                file.write(f"\nAraç {a}:\n")
                file.write(f"Rota: Fabrika -> {' -> '.join(route)} -> Fabrika\n")

                # Her durak için teslimatları göster
                for b in route:
                    deliveries = [(i, x[i, a, b, t].X)
                                  for (i, a2, b2, t2) in x.keys()
                                  if a2 == a and b2 == b and t2 == t and x[i, a2, b2, t2].X > 0.001]
                    if deliveries:
                        file.write(f"  {b} için teslimatlar:\n")
                        for i, amount in deliveries:
                            file.write(f"    Parça {i}: {amount:.2f} birim\n")

        # Günlük özet
        total_deliveries = sum(x[i, a, b, t].X
                               for (i, a, b, t2) in x.keys()
                               if t2 == t)
        file.write(f"\nGün {t} özeti:\n")
        file.write(f"Toplam teslimat sayısı: {total_deliveries:.2f}\n")

        # Aktif araç sayısı
        active_vehicles = sum(1 for a in A if any(w[a, i, j, t].X > 0.5
                                                  for (a2, i, j, t2) in w.keys()
                                                  if a2 == a and t2 == t))
        file.write(f"Kullanılan araç sayısı: {active_vehicles}\n")

        # Toplam maliyet hesapla
        day_transport_cost = sum(G[b] * V[a] * F[(b, a)] * x[i, a, b, t].X
                                 for (i, a, b, t2) in x.keys()
                                 if t2 == t and (b, a) in F)
        day_storage_cost = sum(M[i] * x[i, a, b, t].X
                               for (i, a, b, t2) in x.keys()
                               if t2 == t)
        day_emergency_cost = sum(O[i] * y[i, t].X for i in I)

        file.write(f"Günlük taşıma maliyeti: {day_transport_cost:.2f}\n")
        file.write(f"Günlük depolama maliyeti: {day_storage_cost:.2f}\n")
        file.write(f"Günlük acil durum maliyeti: {day_emergency_cost:.2f}\n")
        file.write(f"Toplam günlük maliyet: {(day_transport_cost + day_storage_cost + day_emergency_cost):.2f}\n")


# Model çözümünden sonra sonuçları dosyaya yazdır
if model.status == GRB.OPTIMAL:
    def get_route_with_deliveries(vehicle, day):
        """Bir aracın belirli bir gündeki rotasını ve teslimatlarını bulur"""
        # Önce teslimat yapıp yapmadığını kontrol et
        has_delivery = False
        for (i, a, b, t) in x.keys():
            if a == vehicle and t == day and x[i, a, b, t].X > 0.001:
                has_delivery = True
                break

        if not has_delivery:
            return None  # Teslimat yapmayan araç için None döndür

        route = []
        current = 'Fabrika'
        visited = {'Fabrika'}

        while True:
            next_stops = [j for j in list(active_locations_by_day[day]) + ['Fabrika']
                          if (vehicle, current, j, day) in w and
                          w[vehicle, current, j, day].X > 0.5 and
                          j not in visited]
            if not next_stops:
                break
            current = next_stops[0]
            if current != 'Fabrika':
                route.append(current)
            visited.add(current)

        return route if route else None


    # Sonuçları dosyaya yazdır
    with open('optimizasyon_sonuclari.txt', 'w', encoding='utf-8') as file:
        file.write("=== OPTİMİZASYON SONUÇLARI ===\n")

        # Karar değişkenlerini yazdır
        print_decision_variables(x, y, z, w, file)

        # Günlük planları yazdır
        write_daily_plans(get_route_with_deliveries, x, z, w, y, I, A, B, T, D, file)

        # Genel özeti yazdır
        file.write("\n=== GENEL ÖZET ===\n")
        total_cost = model.objVal
        file.write(f"Toplam optimizasyon maliyeti: {total_cost:.2f}\n")

        total_transport_cost = sum(G[b] * V[a] * F[(b, a)] * x[i, a, b, t].X
                                   for (i, a, b, t) in x.keys()
                                   if (b, a) in F)
        total_storage_cost = sum(M[i] * x[i, a, b, t].X
                                 for (i, a, b, t) in x.keys())
        total_emergency_cost = sum(O[i] * y[i, t].X
                                   for i in I
                                   for t in T)

        file.write(f"Toplam taşıma maliyeti: {total_transport_cost:.2f}\n")
        file.write(f"Toplam depolama maliyeti: {total_storage_cost:.2f}\n")
        file.write(f"Toplam acil durum maliyeti: {total_emergency_cost:.2f}\n")

    print(f"\nSonuçlar 'optimizasyon_sonuclari.txt' dosyasına yazıldı.")



    def get_route(vehicle, day):
        """Bir aracın belirli bir gündeki rotasını bulur"""
        route = []
        current = 'Fabrika'
        visited = {'Fabrika'}

        while True:
            next_stops = [j for j in list(active_locations_by_day[day]) + ['Fabrika']
                          if (vehicle, current, j, day) in w and
                          w[vehicle, current, j, day].X > 0.5 and
                          j not in visited]
            if not next_stops:
                break
            current = next_stops[0]
            if current != 'Fabrika':
                route.append(current)
            visited.add(current)

        return route


    # Her gün için sonuçları yazdır
    for t in T:
        print(f"\nGün {t} planı:")
        print(f"Bu gün için talepler:")
        day_demands = [(i, b) for (i, b, t2) in demand_tuples if t2 == t]
        for i, b in day_demands:
            print(f"  {b}: Parça {i} - {D[i, b, t]} birim")

        print("\nRota ve teslimatlar:")
        for a in A:
            route = get_route(a, t)
            if route:
                print(f"\nAraç {a}:")
                print(f"Rota: Fabrika -> {' -> '.join(route)} -> Fabrika")

                # Her durak için teslimatları göster
                for b in route:
                    deliveries = [(i, x[i, a, b, t].X)
                                  for (i, a2, b2, t2) in x.keys()
                                  if a2 == a and b2 == b and t2 == t and x[i, a2, b2, t2].X > 0.001]
                    if deliveries:
                        print(f"  {b} için teslimatlar:")
                        for i, amount in deliveries:
                            print(f"    Parça {i}: {amount:.2f} birim")

        # Günlük özet
        total_deliveries = sum(x[i, a, b, t].X
                               for (i, a, b, t2) in x.keys()
                               if t2 == t)
        print(f"\nGün {t} özeti:")
        print(f"Toplam teslimat sayısı: {total_deliveries:.2f}")

        # Aktif araç sayısı
        active_vehicles = sum(1 for a in A if any(w[a, i, j, t].X > 0.5
                                                  for (a2, i, j, t2) in w.keys()
                                                  if a2 == a and t2 == t))
        print(f"Kullanılan araç sayısı: {active_vehicles}")

        # Toplam maliyet hesapla
        day_transport_cost = sum(G[b] * V[a] * F[(b, a)] * x[i, a, b, t].X
                                 for (i, a, b, t2) in x.keys()
                                 if t2 == t and (b, a) in F)
        day_storage_cost = sum(M[i] * x[i, a, b, t].X
                               for (i, a, b, t2) in x.keys()
                               if t2 == t)
        day_emergency_cost = sum(O[i] * y[i, t].X for i in I)

        print(f"Günlük taşıma maliyeti: {day_transport_cost:.2f}")
        print(f"Günlük depolama maliyeti: {day_storage_cost:.2f}")
        print(f"Günlük acil durum maliyeti: {day_emergency_cost:.2f}")
        print(f"Toplam günlük maliyet: {(day_transport_cost + day_storage_cost + day_emergency_cost):.2f}")

    # Genel özet
    print("\nGenel Özet:")
    total_cost = model.objVal
    print(f"Toplam optimizasyon maliyeti: {total_cost:.2f}")

    total_transport_cost = sum(G[b] * V[a] * F[(b, a)] * x[i, a, b, t].X
                               for (i, a, b, t) in x.keys()
                               if (b, a) in F)
    total_storage_cost = sum(M[i] * x[i, a, b, t].X
                             for (i, a, b, t) in x.keys())
    total_emergency_cost = sum(O[i] * y[i, t].X
                               for i in I
                               for t in T)

    print(f"Toplam taşıma maliyeti: {total_transport_cost:.2f}")
    print(f"Toplam depolama maliyeti: {total_storage_cost:.2f}")
    print(f"Toplam acil durum maliyeti: {total_emergency_cost:.2f}")


else:
    print("Optimal çözüm bulunamadı!")
    if model.status == GRB.INFEASIBLE:
        print("\nModel çözülemez durumda.")
        model.computeIIS()
        print("\nÇelişen kısıtlar:")
        for c in model.getConstrs():
            if c.IISConstr:
                print(f"Kısıt {c.ConstrName}")

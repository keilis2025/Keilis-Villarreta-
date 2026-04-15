import os
import platform
import time
import csv
from datetime import datetime
 
MAPA_ANCHO = 5
MAPA_ALTO = 5
ARCHIVO_GUARDAR = "partida_guardada.csv"
 
# ---------- utilidades ----------
def limpiar_pantalla():
    os.system("cls" if platform.system() == "Windows" else "clear")
 
def imprimir_titulo():
    print("====================================")
    print("   Juego del Objetivo PRO 😎        ")
    print("====================================\n")
 
# ---------- jugador ----------
def crear_jugador():
    while True:
        nombre = input("👤 Escribe tu nombre: ").strip()
        if nombre:
            break
        print("⚠️ Debes escribir un nombre.")
 
    return {"nombre": nombre, "x": 0, "y": 0, "vidas": 3, "puntaje": 0}
 
# ---------- niveles ----------
def crear_niveles():
    return [
        {"numero": 1, "objetivo": (4, 4), "trampas": [(1, 2), (2, 3)]},
        {"numero": 2, "objetivo": (3, 1), "trampas": [(1, 1), (4, 0)]},
        {"numero": 3, "objetivo": (0, 4), "trampas": [(2, 2), (3, 3)]}
    ]
 
# ---------- mapa ----------
def crear_mapa(nivel):
    mapa = [["." for _ in range(MAPA_ANCHO)] for _ in range(MAPA_ALTO)]
    x, y = nivel["objetivo"]
    mapa[y][x] = "X"
    for t in nivel["trampas"]:
        mapa[t[1]][t[0]] = "T"
    return mapa
 
def dibujar_mapa(jugador, mapa, nivel):
    limpiar_pantalla()
    imprimir_titulo()
 
    print(f"👤 {jugador['nombre']} | Nivel {nivel['numero']} | ❤️ {jugador['vidas']} | ⭐ {jugador['puntaje']}\n")
 
    for y in range(MAPA_ALTO):
        fila = ""
        for x in range(MAPA_ANCHO):
            if (x, y) == (jugador["x"], jugador["y"]):
                fila += "P "
            else:
                fila += mapa[y][x] + " "
        print(fila)
    print()
 
# ---------- movimiento ----------
def mover_jugador(jugador, comando):
    if comando == "w":
        jugador["y"] -= 1
    elif comando == "s":
        jugador["y"] += 1
    elif comando == "a":
        jugador["x"] -= 1
    elif comando == "d":
        jugador["x"] += 1
 
def validar_posicion(jugador):
    jugador["x"] = max(0, min(MAPA_ANCHO - 1, jugador["x"]))
    jugador["y"] = max(0, min(MAPA_ALTO - 1, jugador["y"]))
 
# ---------- CSV guardar/cargar ----------
def guardar_partida(jugador, nivel):
    with open(ARCHIVO_GUARDAR, "w", newline="", encoding="utf-8") as f:
        writer = csv.writer(f)
        writer.writerow(["nombre","x","y","vidas","puntaje","nivel"])
        writer.writerow([
            jugador["nombre"],
            jugador["x"],
            jugador["y"],
            jugador["vidas"],
            jugador["puntaje"],
            nivel["numero"]
        ])
 
def cargar_partida(niveles):
    if not os.path.exists(ARCHIVO_GUARDAR):
        return None
 
    with open(ARCHIVO_GUARDAR, "r", encoding="utf-8") as f:
        reader = csv.reader(f)
        next(reader)
        datos = next(reader)
 
        nombre, x, y, vidas, puntaje, num = datos
        nivel = next((n for n in niveles if n["numero"] == int(num)), niveles[0])
 
        jugador = {
            "nombre": nombre,
            "x": int(x),
            "y": int(y),
            "vidas": int(vidas),
            "puntaje": int(puntaje)
        }
 
        return jugador, nivel
 
# ---------- ranking ----------
def guardar_estadisticas(jugador):
    archivo = "ranking.csv"
    datos = []
 
    if os.path.exists(archivo):
        with open(archivo, "r", encoding="utf-8") as f:
            reader = csv.reader(f)
            next(reader, None)
            datos = list(reader)
 
    fecha = datetime.now().strftime("%d/%m/%Y %H:%M")
 
    datos.append([
        jugador["nombre"],
        str(jugador["puntaje"]),
        str(jugador["vidas"]),
        fecha
    ])
 
    datos.sort(key=lambda x: int(x[1]), reverse=True)
    datos = datos[:5]
 
    with open(archivo, "w", newline="", encoding="utf-8") as f:
        writer = csv.writer(f)
        writer.writerow(["nombre", "puntaje", "vidas", "fecha"])
        writer.writerows(datos)
 
def mostrar_ranking():
    limpiar_pantalla()
    imprimir_titulo()
 
    archivo = "ranking.csv"
 
    if not os.path.exists(archivo):
        print("😢 No hay ranking todavía.\n")
        input("Enter...")
        return
 
    print("🏆 TOP 5 JUGADORES 🏆\n")
    medallas = ["🥇","🥈","🥉","4️⃣","5️⃣"]
 
    with open(archivo, "r", encoding="utf-8") as f:
        reader = csv.reader(f)
        next(reader)
 
        for i, fila in enumerate(reader):
            print(f"{medallas[i]} {fila[0]} | ⭐ {fila[1]} | ❤️ {fila[2]} | 📅 {fila[3]}")
 
    input("\nPresiona Enter...")
 
# ---------- juego ----------
def juego(cargar=False):
    niveles = crear_niveles()
 
    if cargar:
        data = cargar_partida(niveles)
        if data:
            jugador, nivel = data
        else:
            print("No hay partida guardada.")
            input("Enter...")
            return
    else:
        jugador = crear_jugador()
        nivel = niveles[0]
 
    while True:
        mapa = crear_mapa(nivel)
        dibujar_mapa(jugador, mapa, nivel)
 
        cmd = input("w/a/s/d, guardar, salir: ").lower()
 
        if cmd == "salir":
            print(f"\n👋 {jugador['nombre']}, tu progreso ha sido guardado.")
            guardar_estadisticas(jugador)
            guardar_partida(jugador, nivel)
            time.sleep(1)
            break
 
        if cmd == "guardar":
            guardar_partida(jugador, nivel)
            print("💾 Guardado manual.")
            input("Enter...")
            continue
 
        mover_jugador(jugador, cmd)
        validar_posicion(jugador)
 
        if (jugador["x"], jugador["y"]) in nivel["trampas"]:
            jugador["vidas"] -= 1
            print("💥 Trampa!")
            time.sleep(1)
 
        if jugador["vidas"] <= 0:
            print(f"💀 {jugador['nombre']}, perdiste. Puntaje: {jugador['puntaje']}")
            guardar_estadisticas(jugador)
            time.sleep(2)
            break
 
        if (jugador["x"], jugador["y"]) == nivel["objetivo"]:
            jugador["puntaje"] += 10
            print("🎉 Nivel completado!")
            time.sleep(1)
 
            idx = niveles.index(nivel)
            if idx + 1 < len(niveles):
                nivel = niveles[idx + 1]
                jugador["x"], jugador["y"] = 0, 0
            else:
                print(f"🏆 ¡Felicidades {jugador['nombre']}! Ganaste con {jugador['puntaje']} puntos.")
                guardar_estadisticas(jugador)
                time.sleep(2)
                break
 
# ---------- menú ----------
def menu():
    while True:
        limpiar_pantalla()
        imprimir_titulo()
 
        print("1. Nuevo juego")
        print("2. Cargar partida")
        print("3. Ver ranking")
        print("4. Salir\n")
 
        op = input(">>> ")
 
        if op == "1":
            juego(False)
        elif op == "2":
            juego(True)
        elif op == "3":
            mostrar_ranking()
        elif op == "4":
            break
 
# ---------- main ----------
def main():
    menu()
 
if __name__ == "__main__":
    main()

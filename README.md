# game-traductor
import sounddevice as sd
import scipy.io.wavfile as wav
import speech_recognition as sr
from random import shuffle, choice
import time
import os
import difflib # Para comparar qué tan cerca estuviste de la palabra

# --- CONFIGURACIÓN TÉCNICA ---
FRECUENCIA_MUESTREO = 44100
ARCHIVO_TEMP = "temp_voice.wav"

# --- BASE DE DATOS MASIVA (Simulación de estructura para 100k+) ---
NIVELES_DB = {
    "PRINCIPIANTE": {
        "perro": "dog", "gato": "cat", "sol": "sun", "rojo": "red", "azul": "blue",
        "casa": "house", "pan": "bread", "leche": "milk", "agua": "water", "uno": "one",
        "madre": "mother", "padre": "father", "hermano": "brother", "libro": "book"
    },
    "INTERMEDIO": {
        "ventana": "window", "manzana": "apple", "escuela": "school", "reloj": "clock",
        "zapato": "shoe", "lluvia": "rain", "fuego": "fire", "tierra": "earth",
        "computadora": "computer", "biblioteca": "library", "ciudad": "city", "puente": "bridge"
    },
    "EXPERTO": {
        "escritorio": "desk", "zanahoria": "carrot", "murciélago": "bat", "relámpago": "lightning",
        "edificio": "building", "científico": "scientist", "conocimiento": "knowledge", 
        "desafío": "challenge", "extraordinario": "extraordinary", "ubicuidad": "ubiquity"
    }
}

def limpiar():
    os.system('cls' if os.name == 'nt' else 'clear')

def barra_progreso(actual, total, largo=30):
    porcentaje = actual / total
    bloques = int(largo * porcentaje)
    barra = "█" * bloques + "-" * (largo - bloques)
    return f"|{barra}| {int(porcentaje*100)}%"

def grabar(duracion):
    print("\n" + "┈"*40)
    for i in range(2, 0, -1):
        print(f"       🎤 LISTO EN {i}...", end="\r")
        time.sleep(0.7)
    print("       🎤 ¡HABLA AHORA!      ")
    print("┈"*40)
    
    try:
        grabacion = sd.rec(int(duracion * FRECUENCIA_MUESTREO), 
                           samplerate=FRECUENCIA_MUESTREO, channels=1, dtype='int16')
        sd.wait() 
        wav.write(ARCHIVO_TEMP, FRECUENCIA_MUESTREO, grabacion)
    except Exception as e:
        print(f"⚠️ Error de micro: {e}")

def analizar_voz():
    reconocedor = sr.Recognizer()
    try:
        with sr.AudioFile(ARCHIVO_TEMP) as fuente:
            audio = reconocedor.record(fuente)
        texto = reconocedor.recognize_google(audio, language="en-US")
        return texto.lower().strip(), "OK"
    except:
        return "", "ERROR"

def jugar():
    limpiar()
    print("╔══════════════════════════════════════════════════════════╗")
    print("║        🌟 TRADUCTOR ELITE: ANTI-BRAINROT EDITION 🌟      ║")
    print("╚══════════════════════════════════════════════════════════╝")
    
    print("\n[1] PRINCIPIANTE 🟢 | [2] INTERMEDIO 🟡 | [3] EXPERTO 🔴")
    op = input("\n🕹️ Selecciona Dificultad: ")
    nivel_key = {"1": "PRINCIPIANTE", "2": "INTERMEDIO", "3": "EXPERTO"}.get(op, "PRINCIPIANTE")
    
    # Variables de juego
    puntos, vidas, combo = 0, 5, 0
    objetivo = 300
    palabras_pool = list(NIVELES_DB[nivel_key].items())
    shuffle(palabras_pool)
    stats = {"aciertos": 0, "fallos": 0}

    while vidas > 0 and puntos < objetivo:
        limpiar()
        if not palabras_pool: break
        
        # HUD Superior
        rango = "NOOB" if puntos < 100 else "APRENDIZ" if puntos < 200 else "INTERPRETE"
        print(f"🏆 RANGO: {rango} | XP: {puntos}/{objetivo}")
        print(f"❤️ VIDAS: {'❤️ ' * vidas} | 🔥 COMBO: x{combo}")
        print(barra_progreso(puntos, objetivo))
        print("═"*58)

        es, en = palabras_pool.pop()
        print(f"\n❔ ¿Cómo se dice: 【 {es.upper()} 】?")
        input("\n⌨️  Presiona ENTER para grabar...")

        # Dificultad adaptativa: el tiempo baja si tienes mucho combo
        tiempo = max(1.5, 3.5 - (combo * 0.2))
        grabar(tiempo)
        
        print("\n🔍 Analizando frecuencia...")
        dicho, estado = analizar_voz()

        if estado == "OK":
            # Comparar similitud para dar puntos parciales
            similitud = difflib.SequenceMatcher(None, dicho, en).ratio()
            
            if dicho == en:
                combo += 1
                ganado = 20 + (combo * 5)
                puntos += ganado
                stats["aciertos"] += 1
                print(f"✨ ¡PERFECTO! +{ganado} XP (Exacto: {dicho})")
            elif similitud > 0.7:
                puntos += 10
                print(f"⚠️ ¡CASI! Dijiste '{dicho}', se parece a '{en}'. +10 XP")
            else:
                vidas -= 1
                combo = 0
                stats["fallos"] += 1
                print(f"❌ ERROR. Dijiste '{dicho}', era '{en}'.")
        else:
            vidas -= 1
            combo = 0
            print("🔇 No te escuché. Habla más claro.")
        
        time.sleep(2.5)

    # --- FINAL ---
    limpiar()
    if puntos >= objetivo:
        print("🥇 ¡VICTORIA SUPREMA! 🥇")
        print("Has demostrado una capacidad intelectual superior.")
    else:
        print("💀 GAME OVER - Sigue practicando.")

    print("\n📊 ESTADÍSTICAS DE SESIÓN:")
    print(f"✅ Aciertos: {stats['aciertos']} | ❌ Fallos: {stats['fallos']}")
    
    print("\n" + "!"*58)
    print("   🚫 DI NO AL 67 Y A LOS BRAINROTS 🧠 🚫".center(58))
    print("!"*58 + "\n")

    if os.path.exists(ARCHIVO_TEMP): os.remove(ARCHIVO_TEMP)

if __name__ == "__main__":
    try:
        jugar()
    except KeyboardInterrupt:
        if os.path.exists(ARCHIVO_TEMP): os.remove(ARCHIVO_TEMP)

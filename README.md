# ingenieria-economica-gemini
Programa en Python para evaluación de proyectos de inversión (VPN y TIR) integrado con la IA de Gemini.
"""
Proyecto: Asistente Inteligente de Evaluación de Proyectos de Inversión
Materia: Ingeniería Económica
Licencia: MIT
Conexión: Google Gemini API
"""

import os
try:
    import numpy_financial as npf
except ImportError:
    import os
    os.system('pip install numpy-financial google-genai')
    import numpy_financial as npf

from google import genai
from google.genai import types

def calcular_metricas_financieras(inversion_inicial, flujos, tmar):
    """
    Calcula el VPN y la TIR usando numpy_financial.
    La inversión inicial debe ser un número negativo para el flujo de caja.
    """
    flujos_totales = [inversion_inicial] + flujos
    
    # 1. Cálculo del Valor Presente Neto (VPN)
    vpn = npf.npv(tmar, flujos_totales)
    
    # 2. Cálculo de la Tasa Interna de Retorno (TIR)
    try:
        tir = npf.irr(flujos_totales)
    except Exception:
        tir = None
        
    return vpn, tir

def consultar_consultor_gemini(inversion_inicial, flujos, tmar, vpn, tir, descripcion_proyecto):
    """
    Conecta con la IA Gemini para obtener el análisis económico cualitativo y de riesgos.
    """
    # Inicializa el cliente. Busca automáticamente la variable de entorno GEMINI_API_KEY
    client = genai.Client()
    
    # Formatear la tir para visualización
    tir_texto = f"{tir * 100:.2f}%" if tir is not None else "No calculable"
    
    prompt = f"""
    Actúa como un experto en Ingeniería Económica y consultor financiero senior.
    Evalúa el siguiente proyecto de inversión basado en los datos proporcionados:
    
    - Descripción del Proyecto: {descripcion_proyecto}
    - Inversión Inicial: ${abs(inversion_inicial):,.2f}
    - Flujos de Efectivo Netos Anuales: {[f"${x:,.2f}" for x in flujos]}
    - Tasa Mínima Aceptable de Rendimiento (TMAR): {tmar * 100:.2f}%
    
    Resultados Financieros Obtenidos:
    - Valor Presente Neto (VPN): ${vpn:,.2f}
    - Tasa Interna de Retorno (TIR): {tir_texto}
    
    Por favor, genera un informe ejecutivo que incluimos:
    1. Dictamen Técnico (¿Se acepta o se rechaza el proyecto según los criterios de Ingeniería Económica?).
    2. Análisis cualitativo de los flujos y rentabilidad.
    3. Tres riesgos críticos asociados a este tipo de proyecto y cómo mitigarlos frente a la volatilidad económica actual.
    """
    
    print("\n[INFO] Conectando con Gemini para el análisis experto...")
    
    response = client.models.generate_content(
        model='gemini-2.5-flash',
        contents=prompt,
    )
    
    return response.text

def main():
    print("==========================================================")
    print("  ASISTENTE DE INGENIERÍA ECONÓMICA POTENCIADO POR GEMINI ")
    print("==========================================================\n")
    
    # Datos de entrada del problema de la vida real
    descripcion = input("Breve descripción del proyecto (ej. Automatización de línea de empaque): ")
    inversion = float(input("Ingrese el monto de la Inversión Inicial (Ej: 50000): "))
    inversion_negativa = -abs(inversion) # Aseguramos que sea salida de dinero
    
    años = int(input("¿A cuántos años está proyectado el proyecto?: "))
    flujos = []
    for i in range(años):
        flujo = float(input(f"Ingrese el flujo de efectivo neto para el Año {i+1}: "))
        flujos.append(flujo)
        
    tmar_porcentaje = float(input("Ingrese la TMAR deseada en porcentaje (Ej: 15 para 15%): "))
    tmar = tmar_porcentaje / 100

    # Procesamiento Matemático (Ingeniería Económica pura)
    vpn, tir = calcular_metricas_financieras(inversion_negativa, flujos, tmar)
    
    print("\n----------------------------------------------------------")
    print("RESULTADOS MATEMÁTICOS:")
    print(f"-> Valor Presente Neto (VPN): ${vpn:,.2f}")
    if tir is not None:
        print(f"-> Tasa Interna de Retorno (TIR): {tir * 100:.2f}%")
    else:
        print("-> Tasa Interna de Retorno (TIR): No se pudo calcular.")
    print("----------------------------------------------------------")

    # Integración con Inteligencia Artificial (Gemini)
    # Nota: Asegúrate de tener tu API KEY configurada en tu terminal: export GEMINI_API_KEY="tu_clave"
    if not os.environ.get("GEMINI_API_KEY"):
        print("\n[ALERTA] No se detectó la variable de entorno GEMINI_API_KEY.")
        api_key_manual = input("Por favor, introduce tu Gemini API Key para continuar: ")
        os.environ["GEMINI_API_KEY"] = api_key_manual

    try:
        analisis_ia = consultar_consultor_gemini(inversion_negativa, flujos, tmar, vpn, tir, descripcion)
        print("\n==========================================================")
        print("REPORTE EJECUTIVO GENERADO POR GEMINI IA:")
        print("==========================================================")
        print(analisis_ia)
    except Exception as e:
        print(f"\n[ERROR] Ocurrió un problema al conectar con Gemini: {e}")

if __name__ == "__main__":
    main()

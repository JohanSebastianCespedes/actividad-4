import streamlit as st
import pandas as pd
import polars as pl
import matplotlib.pyplot as plt

# Configuración de página estilo Dark Mode / Trading Dashboard
st.set_page_config(page_title="Dashboard de Inversión - Coink", page_icon="📈", layout="wide")

st.title("📈 Dashboard de Análisis y Performance de Inversión - Coink")
st.markdown("Módulo analítico inspirado en gestión de portafolios y volumen de transacciones para calificar el rendimiento de los usuarios.")

# Cargar datos eficientemente con Polars y pasarlos a Pandas
@st.cache_data
def cargar_datos():
    df_pl = pl.read_csv("depositos_oinks.csv")
    return df_pl.to_pandas()

try:
    df = cargar_datos()

    # Agrupamos los datos simulando métricas de portafolio (Volumen y Frecuencia de Órdenes)
    resumen = df.groupby('user_id').agg(
        capital_total=('operation_value', 'sum'),
        frecuencia_operaciones=('operation_value', 'count'),
        ticket_promedio=('operation_value', 'mean')
    ).reset_index()

    # Normalización para calcular el índice IRAI (Escala de 0 a 100)
    max_capital = resumen['capital_total'].max()
    max_freq = resumen['frecuencia_operaciones'].max()

    resumen['norm_capital'] = resumen['capital_total'] / max_capital
    resumen['norm_freq'] = resumen['frecuencia_operaciones'] / max_freq
    
    # Cálculo del IRAI (60% Volumen de Capital, 40% Frecuencia de Operación)
    resumen['IRAI'] = ((resumen['norm_capital'] * 0.6) + (resumen['norm_freq'] * 0.4)) * 100

    def clasificar_trader(irai):
        if irai >= 40:
            return "Trader Pro / Ballena ⭐"
        elif irai >= 15:
            return "Inversionista Constante 🟢"
        else:
            return "Trader Junior 🟠"

    resumen['Perfil'] = resumen['IRAI'].apply(clasificar_trader)
    resumen = resumen.sort_values(by='IRAI', ascending=False)

    # Panel de Métricas Clave (KPIs estilo Terminal de Trading)
    col1, col2, col3 = st.columns(3)
    col1.metric("Total Inversionistas", len(resumen))
    col2.metric("Volumen Global Operado", f"${resumen['capital_total'].sum():,.2f}")
    col3.metric("IRAI Promedio del Portafolio", f"{resumen['IRAI'].mean():.2f} pts")

    st.subheader("📊 Tabla de Calificación de Actividad y Performance")
    st.dataframe(resumen[['user_id', 'capital_total', 'frecuencia_operaciones', 'ticket_promedio', 'IRAI', 'Perfil']].head(15), use_container_width=True)

    # Gráfica de Rendimiento con Matplotlib (Diseño limpio acorde al análisis técnico)
    st.subheader("📉 Top 10 Portafolios con Mayor Rendimiento (IRAI)")
    top10 = resumen.head(10)
    
    fig, ax = plt.subplots(figsize=(10, 4))
    ax.barh(top10['user_id'].str[:8] + "...", top10['IRAI'], color='#f39c12')
    ax.set_xlabel("Puntaje IRAI (0 - 100)")
    ax.set_ylabel("ID de Usuario / Wallet")
    ax.set_title("Ranking de Rendimiento de Usuarios en la Red")
    ax.set_xlim(0, 100)
    ax.invert_yaxis()
    ax.grid(axis='x', linestyle='--', alpha=0.3)
    
    st.pyplot(fig)

except Exception as e:
    st.error(f"Error procesando el dataset de transacciones: {e}")

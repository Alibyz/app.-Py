# app.-Pyimport streamlit as st
import pandas as pd
import matplotlib.pyplot as plt

st.set_page_config(page_title="Sanal Köpek Yarışı Analiz", layout="centered")

st.title("Sanal Köpek Yarışı Analiz Botu")
st.write("CSV dosyanı yükle, sistem senin için kazanan oranları analiz etsin!")

uploaded_file = st.file_uploader("CSV dosyasını yükle", type=["csv"])

if uploaded_file:
    df = pd.read_csv(uploaded_file)

    # Oran aralığına göre etiketleme
    bins = [0, 2.0, 3.5, 5.0, 7.5, 10.0, 20.0]
    labels = ['<2.0', '2.0-3.5', '3.5-5.0', '5.0-7.5', '7.5-10', '10+']
    df['odds_range'] = pd.cut(df['winner_odds'], bins=bins, labels=labels)

    st.subheader("Kazanan Oran Aralığı Dağılımı")
    odds_dist = df['odds_range'].value_counts(normalize=True).sort_index() * 100
    st.bar_chart(odds_dist)

    st.write("**Kazanan oran yüzdeleri:**")
    st.dataframe(odds_dist.round(2))

    # Favori kazandı mı analizi
    def is_fav(row):
        all_odds = [row[f'dog{i}_odds'] for i in range(1, 7)]
        return row['winner_odds'] == min(all_odds)

    df['favorite_won'] = df.apply(is_fav, axis=1)
    fav_rate = df['favorite_won'].mean() * 100

    st.subheader("Favori Kazanma Oranı")
    st.metric(label="Favori kazanma yüzdesi", value=f"%{fav_rate:.2f}")

    # Sistem önerisi
    best_range = odds_dist.idxmax()
    st.success(f"Öneri: En çok kazanan oran aralığı **'{best_range}'** — bu aralığı takip etmeni öneririm.")

else:
    st.warning("Başlamak için yarış sonuçlarını içeren bir CSV dosyası yüklemelisin.")

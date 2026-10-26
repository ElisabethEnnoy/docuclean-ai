import streamlit as st
import openai
import pypdf

st.set_page_config(page_title="DocuClean AI - Analiză Contracte", page_icon="📄", layout="centered")

# === CONFIGURARE AMBALAJ PRODUS (PUNE LINK-UL TĂU AICI) ===
LEMON_SQUEEZY_LINK = https://docuclean.lemonsqueezy.com/checkout/buy/76297748-b6f0-4c5c-b0e0-dbe3560a302b

st.title("📄 DocuClean AI")
st.subheader("Analizează orice contract în 10 secunde. Descoperă riscurile ascunse.")

# Secțiune Hero / Abonament
st.info("💡 **Ai nevoie de acces complet?** Abonează-te la **DocuClean Pro (19€/lună)** pentru analize nelimitate ale contractelor tale.")
st.markdown(f"[👉 **Click Aici pentru Abonare / Acces Pro**]({LEMON_SQUEEZY_LINK})", unsafe_allow_dict=True)

st.markdown("---")

# Căsuță securizată pentru API Key (sau cheia de acces generată după plată)
api_key = st.text_input("Introdu OpenAI API Key (sau cheia de acces primită):", type="password")

uploaded_file = st.file_uploader("Încarcă contractul în format PDF", type=["pdf"])

def extract_text_from_pdf(pdf_file):
    reader = pypdf.PdfReader(pdf_file)
    text = ""
    for page in reader.pages:
        text += page.extract_text() or ""
    return text

def analyze_contract(text, api_key):
    client = openai.OpenAI(api_key=api_key)
    prompt = f"""
    Ești un expert juridic specializat în analiza contractelor pentru firme mici și agenți imobiliari.
    Analizează textul următorului contract și oferă un răspuns structurat în limba română:
    
    1. 📌 REZUMAT EXECUTIV (max 3 fraze simple).
    2. ⚠️ CLAUZE DE RISC & PENALITĂȚI ASCUNSE (evidențiază tot ce poate costa bani sau aduce probleme legale).
    3. ✅ OBLIGAȚII CHEIE (ce trebuie să facă utilizatorul concret).

    Text Contract:
    {text[:12000]}
    """
    
    response = client.chat.completions.create(
        model="gpt-4o-mini",
        messages=[{"role": "user", "content": prompt}],
        temperature=0.2
    )
    return response.choices[0].message.content

if uploaded_file is not None:
    if not api_key:
        st.warning("⚠️ Te rugăm să introduci cheia ta API pentru a efectua analiza.")
    else:
        with st.spinner("Se extrage textul și se analizează clauzele..."):
            try:
                contract_text = extract_text_from_pdf(uploaded_file)
                if len(contract_text.strip()) < 50:
                    st.error("Nu s-a putut citi textul din PDF. Asigură-te că nu este o imagine scanată fără OCR.")
                else:
                    report = analyze_contract(contract_text, api_key)
                    st.success("Analiză finalizată!")
                    st.markdown("---")
                    st.markdown(report)
                    st.download_button("Descarcă Raportul (TXT)", report, file_name="Raport_DocuClean.txt")
            except Exception as e:
                st.error(f"A apărut o eroare la procesare: {e}")

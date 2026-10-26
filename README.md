import streamlit as st
import openai
import pypdf

st.set_page_config(page_title="DocuClean AI - Analiză Contracte", page_icon="📄", layout="centered")

# === https://docuclean.lemonsqueezy.com/checkout/buy/76297748-b6f0-4c5c-b0e0-dbe3560a302b ===
LEMON_SQUEEZY_LINK = https://docuclean.lemonsqueezy.com/checkout/buy/76297748-b6f0-4c5c-b0e0-dbe3560a302b

st.title("📄 DocuClean AI")
st.subheader("Analizează orice contract în 10 secunde. Descoperă riscurile ascunse.")

# Secțiune Abonament
st.info("💡 **Acces Nelimitat:** Abonează-te la **DocuClean Pro (19€/lună)** pentru a analiza contracte instant fără griji.")
st.markdown(f"[👉 **Obține Acces Pro pe Lemon Squeezy**]({LEMON_SQUEEZY_LINK})", unsafe_allow_html=True)

st.markdown("---")

uploaded_file = st.file_uploader("Încarcă contractul în format PDF", type=["pdf"])

def extract_text_from_pdf(pdf_file):
    reader = pypdf.PdfReader(pdf_file)
    text = ""
    for page in reader.pages:
        text += page.extract_text() or ""
    return text

def analyze_contract(text):
    # Folosește cheia secretă configurată în Streamlit Cloud
    client = openai.OpenAI(api_key=st.secrets["OPENAI_API_KEY"])
    
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
    with st.spinner("Se extrage textul și se analizează clauzele..."):
        try:
            contract_text = extract_text_from_pdf(uploaded_file)
            if len(contract_text.strip()) < 50:
                st.error("Nu s-a putut citi textul din PDF. Asigură-te că nu este o imagine scanată fără OCR.")
            else:
                report = analyze_contract(contract_text)
                st.success("Analiză finalizată!")
                st.markdown("---")
                st.markdown(report)
                st.download_button("Descarcă Raportul (TXT)", report, file_name="Raport_DocuClean.txt")
        except Exception as e:
            st.error(f"A apărut o eroare la procesare: {e}")

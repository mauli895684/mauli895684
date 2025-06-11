import streamlit as st
import pdfplumber
import io
import datetime

st.title("Bank Statement Average Balance Calculator")

dates_required = [5, 10, 15, 25, 30]

uploaded_file = st.file_uploader("Upload your bank statement (PDF)", type="pdf")

def extract_balances(pdf_bytes, dates_required):
    date_balance = {}
    with pdfplumber.open(io.BytesIO(pdf_bytes)) as pdf:
        for page in pdf.pages:
            text = page.extract_text()
            if text:
                for line in text.split('\n'):
                    # Simple pattern: Date at start, then ... then Balance at end
                    parts = line.strip().split()
                    if len(parts) >= 2:
                        try:
                            # Try to parse date (change format as per your statement)
                            dt = datetime.datetime.strptime(parts[0], "%d-%m-%Y")
                            balance = float(parts[-1].replace(',', ''))
                            date_balance[dt.day] = balance
                        except:
                            pass
    # Now, for required dates, get balance or previous available
    balances = []
    last_balance = 0
    for d in dates_required:
        if d in date_balance:
            last_balance = date_balance[d]
            balances.append(last_balance)
        else:
            balances.append(last_balance)
    return balances

if uploaded_file is not None:
    balances = extract_balances(uploaded_file.read(), dates_required)
    st.write("Balances for dates:", dict(zip(dates_required, balances)))
    avg = sum(balances) / len(balances)
    st.success(f"Average Balance: ₹{avg:,.2f}")

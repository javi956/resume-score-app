import docx
import os
import fitz  # PyMuPDF
import streamlit as st
import pandas as pd

def extract_text_from_docx(file_path):
    doc = docx.Document(file_path)
    return '\n'.join([p.text for p in doc.paragraphs])

def extract_text_from_pdf(file_path):
    text = ""
    with fitz.open(file_path) as doc:
        for page in doc:
            text += page.get_text()
    return text

def score_resume(text, criteria):
    score = 0
    # Score by skills
    for skill in criteria.get("skills", []):
        if skill.lower() in text.lower():
            score += 10
    # Score by education
    for degree in criteria.get("education", []):
        if degree.lower() in text.lower():
            score += 20
    # Score by keywords
    for keyword in criteria.get("keywords", []):
        if keyword.lower() in text.lower():
            score += 5
    return score

st.title("Resume Scorer App")

uploaded_files = st.file_uploader("Upload Resumes (.pdf or .docx)", accept_multiple_files=True, type=["pdf", "docx"])
skills = st.text_input("Required Skills (comma-separated)")
education = st.text_input("Required Education Levels (comma-separated)")
keywords = st.text_input("Keywords (comma-separated)")

if st.button("Score Resumes") and uploaded_files:
    criteria = {
        "skills": [s.strip() for s in skills.split(",") if s.strip()],
        "education": [e.strip() for e in education.split(",") if e.strip()],
        "keywords": [k.strip() for k in keywords.split(",") if k.strip()]
    }
    results = []
    for uploaded_file in uploaded_files:
        if uploaded_file.name.endswith(".docx"):
            text = extract_text_from_docx(uploaded_file)
        elif uploaded_file.name.endswith(".pdf"):
            with open("temp.pdf", "wb") as f:
                f.write(uploaded_file.read())
            text = extract_text_from_pdf("temp.pdf")
            os.remove("temp.pdf")
        else:
            continue

        score = score_resume(text, criteria)
        results.append((uploaded_file.name, score))

    sorted_results = sorted(results, key=lambda x: x[1], reverse=True)
    df = pd.DataFrame(sorted_results, columns=["Filename", "Score"])
    st.dataframe(df)

    csv = df.to_csv(index=False).encode('utf-8')
    st.download_button(
        label="Download Results as CSV",
        data=csv,
        file_name="resume_scores.csv",
        mime="text/csv"
    )

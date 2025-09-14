import streamlit as st
import sqlite3
import os
import pandas as pd

# Database and file storage setup
DB_NAME = "students.db"
DOCS_DIR = "student_docs"
os.makedirs(DOCS_DIR, exist_ok=True)

def init_db():
    conn = sqlite3.connect(DB_NAME)
    cur = conn.cursor()
    cur.execute('''
        CREATE TABLE IF NOT EXISTS students (
            roll_no TEXT PRIMARY KEY,
            name TEXT NOT NULL,
            document_path TEXT
        )
    ''')
    conn.commit()
    conn.close()

def add_student(roll_no, name, document):
    conn = sqlite3.connect(DB_NAME)
    cur = conn.cursor()
    try:
        cur.execute("INSERT INTO students VALUES (?, ?, ?)", (roll_no, name, document))
        conn.commit()
        st.success("✅ Student added successfully")
    except sqlite3.IntegrityError:
        st.error("❌ Roll Number already exists")
    conn.close()

def fetch_students(order_by="roll_no"):
    conn = sqlite3.connect(DB_NAME)
    df = pd.read_sql_query(f"SELECT * FROM students ORDER BY {order_by}", conn)
    conn.close()
    return df

def export_data(df):
    return df.to_csv(index=False).encode("utf-8")

# Streamlit UI
st.set_page_config(page_title="School Student Management", layout="wide")
st.title("🏫 School Student Management")

# Initialize DB
init_db()

# Sidebar: Add new student
st.sidebar.header("➕ Add Student")
roll_no = st.sidebar.text_input("Roll Number")
name = st.sidebar.text_input("Name")
uploaded_file = st.sidebar.file_uploader("Upload Document", type=["pdf", "jpg", "png", "docx"])

if st.sidebar.button("Add Student"):
    if not roll_no or not name:
        st.sidebar.error("Roll No and Name are required")
    else:
        doc_path = ""
        if uploaded_file is not None:
            doc_path = os.path.join(DOCS_DIR, f"{roll_no}_{uploaded_file.name}")
            with open(doc_path, "wb") as f:
                f.write(uploaded_file.getbuffer())
        add_student(roll_no, name, doc_path)

# Main: Show students
st.subheader("📋 Student Records")

sort_by = st.selectbox("Sort by", ["roll_no", "name"], index=0)
students_df = fetch_students(order_by=sort_by)

if not students_df.empty:
    st.dataframe(students_df, use_container_width=True)

    # Download option
    csv = export_data(students_df)
    st.download_button("📥 Download as CSV", csv, "students.csv", "text/csv")
else:
    st.info("No students found. Add some from the sidebar.")


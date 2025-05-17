# PROJECT1
import streamlit as st
import zipfile
import tempfile
import os
import fitz  # PyMuPDF
import re
import nltk
from sentence_transformers import SentenceTransformer, util

# Ensure NLTK stopwords are available
try:
    nltk.data.find('corpora/stopwords')
except LookupError:
    nltk.download('stopwords')

from nltk.corpus import stopwords
stop_words = set(stopwords.words('english'))

# Load sentence transformer model
model = SentenceTransformer('all-MiniLM-L6-v2')

# Job role database
job_roles = {
    "Data Scientist": {
        "keywords": ["python", "machine learning", "statistics", "pandas", "numpy", "regression", "classification", "matplotlib", "r", "scikit-learn", "data analysis", "data visualization", "deep learning"],
        "description": "Analyze data, build machine learning models, and visualize data insights."
    },
    "Full Stack Developer": {
        "keywords": ["react", "node.js", "html", "css", "javascript", "mongodb", "express", "frontend", "backend", "api", "typescript", "spring boot", "jsp", "rest api", "microservices", "sql"],
        "description": "Develop both client-side and server-side software including APIs, databases, and user interfaces."
    },
    "DevOps Engineer": {
        "keywords": ["docker", "kubernetes", "ansible", "terraform", "ci/cd", "jenkins", "aws", "linux", "scripting", "infrastructure as code", "monitoring", "cloud"],
        "description": "Automate infrastructure, manage deployments, and monitor cloud services."
    },
    "Java Developer": {
        "keywords": ["java", "spring", "hibernate", "android", "backend", "api", "maven", "junit", "rest", "microservices"],
        "description": "Enterprise apps, Android, backend APIs."
    },
    "Python Developer": {
        "keywords": ["python", "django", "flask", "automation", "data science", "pandas", "numpy", "ai", "ml"],
        "description": "Web apps, automation, data science, AI/ML."
    },
    "JavaScript Developer": {
        "keywords": ["javascript", "react", "node.js", "vue", "angular", "frontend", "backend", "express"],
        "description": "Web development (frontend/backend with Node.js)."
    },
    "TypeScript Developer": {
        "keywords": ["typescript", "react", "angular", "node.js", "frontend", "backend"],
        "description": "Typed JavaScript; modern frontend/backend development."
    },
    "C#/.NET Developer": {
        "keywords": ["c#", ".net", "asp.net", "windows", "backend", "api", "mvc"],
        "description": "Windows apps, enterprise apps, backend APIs."
    },
    "C++ Developer": {
        "keywords": ["c++", "game development", "systems programming", "embedded"],
        "description": "Game development, systems programming, embedded software."
    },
    "Go (Golang) Developer": {
        "keywords": ["go", "golang", "concurrency", "backend", "cloud"],
        "description": "High-performance backend services, cloud applications."
    },
    "Rust Developer": {
        "keywords": ["rust", "systems programming", "performance", "blockchain"],
        "description": "Systems programming, blockchain, performance-critical applications."
    },
    "PHP Developer": {
        "keywords": ["php", "laravel", "wordpress", "web development"],
        "description": "Web development, CMS platforms like WordPress."
    },
    "Ruby Developer": {
        "keywords": ["ruby", "rails", "web applications"],
        "description": "Web applications using Ruby on Rails."
    },
    "Kotlin Developer": {
        "keywords": ["kotlin", "android", "backend"],
        "description": "Android development, backend services."
    },
    "Swift Developer": {
        "keywords": ["swift", "ios", "macos", "xcode"],
        "description": "iOS/macOS application development."
    },
   "R Developer": {
        "keywords": ["r", "data analysis", "statistics", "ggplot2"],
        "description": "Data analysis, statistical computing."
    },
    "Frontend Developer": {
        "keywords": ["html", "css", "javascript", "react", "vue", "angular", "ui", "ux"],
        "description": "UI/UX, HTML/CSS/JS, frameworks like React/Vue."
    },
    "Backend Developer": {
        "keywords": ["node.js", "express", "django", "flask", "api", "sql", "mongodb", "server"],
        "description": "Server logic, databases, APIs."
    },
    "Mobile App Developer": {
        "keywords": ["android", "ios", "kotlin", "swift", "flutter", "react native"],
        "description": "Android (Java/Kotlin), iOS (Swift), or cross-platform (Flutter/React Native)."
    },
    "Desktop App Developer": {
        "keywords": ["electron", "qt", "wpf", "winforms", "macos"],
        "description": "Applications for Windows/macOS/Linux."
    },
    "Embedded Systems Developer": {
        "keywords": ["embedded", "firmware", "microcontroller", "c", "c++", "iot"],
        "description": "Software for hardware devices, IoT."
       },
    "Machine Learning Engineer": {
        "keywords": ["machine learning", "tensorflow", "pytorch", "ai", "ml", "data pipeline"],
        "description": "Build ML models, data pipelines, AI features."
    },
    "Data Engineer": {
        "keywords": ["etl", "big data", "hadoop", "spark", "airflow", "sql"],
        "description": "ETL pipelines, big data systems, database design."
    },
    "Cloud Engineer": {
        "keywords": ["aws", "azure", "gcp", "cloud architecture", "terraform", "devops"],
        "description": "Design cloud architecture (AWS, Azure, GCP)."
    },
    "Site Reliability Engineer (SRE)": {
        "keywords": ["uptime", "monitoring", "scalability", "infrastructure"],
        "description": "Maintain uptime, reliability, and scalability."
    },
    "Security Engineer": {
        "keywords": ["security", "penetration testing", "vulnerability", "audit"],
        "description": "Secure software, conduct audits, penetration testing."
    },
    "QA/Test Automation Engineer": {
        "keywords": ["qa", "testing", "selenium", "test automation", "ci/cd"],
        "description": "Automated and manual testing, quality assurance pipelines."
    },

  "Game Developer": {
        "keywords": ["unity", "unreal", "game development", "c++", "c#"],
        "description": "Develop games using Unity, Unreal Engine, etc."
    },
    "Blockchain Developer": {
        "keywords": ["blockchain", "solidity", "smart contracts", "dapps", "ethereum"],
        "description": "Build decentralized apps (DApps), smart contracts (Solidity, Rust)."
    },
    "AR/VR Developer": {
        "keywords": ["ar", "vr", "unity", "unreal", "immersive"],
        "description": "Build immersive applications (Unity, Unreal, C++)."
    },
    "Firmware Developer": {
        "keywords": ["firmware", "microcontroller", "embedded", "c"],
        "description": "Write low-level software for microcontrollers and electronic devices."
    },
    "Software Architect": {
        "keywords": ["architecture", "design patterns", "system design"],
        "description": "Design high-level software structures and system interactions."
    },
    "Technical Lead (Tech Lead)": {
        "keywords": ["lead", "mentoring", "architecture", "technical direction"],
        "description": "Guides technical direction, mentors developers."
    },
        "Product Manager": {
        "keywords": ["product", "requirements", "user stories", "roadmap"],
        "description": "Defines product goals, works with developers to build features."
    },
    "Scrum Master / Agile Coach": {
        "keywords": ["agile", "scrum", "facilitation", "sprints"],
        "description": "Facilitates Agile development process."
    },
    "UI/UX Designer": {
        "keywords": ["ui", "ux", "design", "wireframe", "prototype"],
        "description": "Designs user interfaces and experience flows."
    },
    "Technical Writer": {
        "keywords": ["documentation", "writing", "technical content"],
        "description": "Creates technical documentation and content."
    }
}

def extract_text_from_pdf(path):
    doc = fitz.open(path)
    text = ""
    for page in doc:
        text += page.get_text()
    text = text.lower()
    return re.sub(r'\s+', ' ', text)

def preprocess_text(text):
    words = re.findall(r'\b[a-z]+\b', text)
    return ' '.join([w for w in words if w not in stop_words])
def semantic_match(resume_text, role_info):
    resume_emb = model.encode(resume_text, convert_to_tensor=True)
    role_emb = model.encode(role_info["description"], convert_to_tensor=True)
    similarity = util.pytorch_cos_sim(resume_emb, role_emb).item()
    matched = [kw for kw in role_info["keywords"] if kw in resume_text]
    missing = [kw for kw in role_info["keywords"] if kw not in resume_text]
    return similarity, matched, missing

def get_feedback(score, matched, missing, role):
    if score < 0.4:
        return f"❌ Low relevance to *{role}*. Consider adding: {', '.join(missing[:5])}."
    elif score < 0.7:
        return f"⚠️ Partial match for *{role}*. Improve by highlighting: {', '.join(missing[:3])}."
    else:
        return f"✅ Strong fit for *{role}* with skills: {', '.join(matched[:5])}."

def calculate_grade(score):
    if score >= 0.7:
        return "A (Excellent)"
    elif score >= 0.5:
        return "B (Good)"
    elif score >= 0.3:
        return "C (Average)"
    else:
        return "D (Needs Improvement)"
# Streamlit UI
st.set_page_config("Resume Analyzer - Bulk ZIP Upload", layout="centered")
st.title("📁 AI Resume Analyzer (Bulk ZIP Upload)")
st.write("Upload a .zip file containing multiple PDF resumes and get matching scores by role!")

zip_file = st.file_uploader("Upload ZIP File of Resumes", type="zip")
desired_role = st.text_input("🎯 Enter Desired Role (e.g., Data Scientist)")

if zip_file and desired_role:
    if desired_role not in job_roles:
        st.error("❗ Role not found. Please choose a valid predefined role.")
    else:
        role_info = job_roles[desired_role]

   with tempfile.TemporaryDirectory() as tmp_dir:
            with zipfile.ZipFile(zip_file, "r") as z:
                z.extractall(tmp_dir)

   pdf_files = [f for f in os.listdir(tmp_dir) if f.lower().endswith(".pdf")]
            if not pdf_files:
                st.warning("No PDF files found in the uploaded ZIP.")
            else:
                st.subheader(f"📊 Resume Matching for Role: {desired_role}")
                results = []
                for file_name in pdf_files:
                    path = os.path.join(tmp_dir, file_name)
                    raw_text = extract_text_from_pdf(path)
                    resume_text = preprocess_text(raw_text)
                    score, matched, missing = semantic_match(resume_text, role_info)

   candidate_name = file_name.replace(".pdf", "").replace("_", " ").title()
                    results.append({
                        "name": candidate_name,
                        "score": score,
                        "matched_keywords": matched,
                        "missing_keywords": missing
                    })

   results.sort(key=lambda x: x["score"], reverse=True)

   for res in results:
                    st.markdown(f"### 👤 {res['name']}")
                    st.write(f"*Match Score:* {round(res['score'] * 100, 2)}%")
                    st.write(f"*Resume Grade:* {calculate_grade(res['score'])}")
                    st.write(f"✅ *Matched Skills:* {', '.join(res['matched_keywords'][:7]) or 'None'}")
                    st.write(f"❌ *Missing Skills:* {', '.join(res['missing_keywords'][:7]) or 'None'}")
                    st.info(get_feedback(res['score'], res['matched_keywords'], res['missing_keywords'], desired_role))
                    st.markdown("---")
else:
    st.info("Please upload a .zip file and enter a valid desired role.")










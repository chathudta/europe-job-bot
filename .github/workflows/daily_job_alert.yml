import requests
import smtplib
from email.mime.multipart import MIMEMultipart
from email.mime.text import MIMEText
from datetime import datetime
import os

# --- CONFIGURATION ---
# 1. Search Parameters
KEYWORDS = [
    "ERP Implementation Lead",
    "Senior Application Consultant",
    "Application Manager"
]
COUNTRIES = ["Germany", "Netherlands", "Ireland"]

# 2. Email Credentials (Use Gmail App Password, NOT your regular password)
SENDER_EMAIL = os.environ.get("SENDER_EMAIL", "chathu.dta@gmail.com")
APP_PASSWORD = os.environ.get("EMAIL_APP_PASSWORD", "YOUR_16_DIGIT_APP_PASSWORD")
RECEIVER_EMAIL = os.environ.get("RECEIVER_EMAIL", "chathu.dta@gmail.com")

# 3. Optional: Adzuna API Keys (Free tier at developer.adzuna.com - great for DE, NL, IE/UK)
# If left empty, the script uses the free Remotive/Arbeitnow public APIs + generates direct search links.
ADZUNA_APP_ID = os.environ.get("ADZUNA_APP_ID", "")
ADZUNA_APP_KEY = os.environ.get("ADZUNA_APP_KEY", "")


def fetch_arbeitnow_jobs():
    """Fetches European/German tech & visa-sponsored jobs from Arbeitnow public API (No API key needed)."""
    url = "https://www.arbeitnow.com/api/job-board-api"
    matched_jobs = []
    try:
        resp = requests.get(url, timeout=15)
        if resp.status_code == 200:
            data = resp.json().get("data", [])
            for job in data:
                title = job.get("title", "")
                desc = job.get("description", "").lower()
                tags = [t.lower() for t in job.get("tags", [])]
                
                # Check if title matches ERP / Application / Consultant / Implementation Lead
                title_lower = title.lower()
                is_relevant = any(term in title_lower for term in [
                    "erp", "application consultant", "implementation lead", 
                    "application manager", "sap", "dynamics", "enterprise application"
                ])
                
                if is_relevant:
                    visa_sponsored = "visa" in desc or "relocation" in desc or "visa sponsorship" in tags
                    matched_jobs.append({
                        "title": title,
                        "company": job.get("company_name", "N/A"),
                        "location": job.get("location", "Germany / Europe"),
                        "url": job.get("url", ""),
                        "visa_tag": "Visa/Relocation Mentioned" if visa_sponsored else "Check Details",
                        "source": "Arbeitnow (EU)"
                    })
    except Exception as e:
        print(f"Error fetching Arbeitnow: {e}")
    return matched_jobs


def fetch_adzuna_jobs():
    """Fetches jobs from Adzuna API for Germany (de) and Netherlands (nl) if API keys are provided."""
    if not ADZUNA_APP_ID or not ADZUNA_APP_KEY:
        return []
    
    matched_jobs = []
    country_codes = {"Germany": "de", "Netherlands": "nl"}
    
    for country_name, code in country_codes.items():
        for kw in KEYWORDS:
            url = f"https://api.adzuna.com/v1/api/jobs/{code}/search/1"
            params = {
                "app_id": ADZUNA_APP_ID,
                "app_key": ADZUNA_APP_KEY,
                "results_per_page": 10,
                "what": kw,
                "max_days_old": 2  # Only jobs posted in the last 48 hours
            }
            try:
                resp = requests.get(url, params=params, timeout=15)
                if resp.status_code == 200:
                    results = resp.json().get("results", [])
                    for item in results:
                        matched_jobs.append({
                            "title": item.get("title", ""),
                            "company": item.get("company", {}).get("display_name", "N/A"),
                            "location": f"{item.get('location', {}).get('display_name', '')} ({country_name})",
                            "url": item.get("redirect_url", ""),
                            "visa_tag": "Direct Listing",
                            "source": f"Adzuna ({country_name})"
                        })
            except Exception as e:
                print(f"Error fetching Adzuna for {country_name}: {e}")
    return matched_jobs


def build_direct_search_links():
    """Generates 1-click daily filtered search URLs for LinkedIn & StepStone (Last 24 hours)."""
    links = [
        ("LinkedIn - Germany (ERP / App Consultant - Past 24h)", 
         "https://www.linkedin.com/jobs/search/?keywords=ERP%20Implementation%20OR%20Application%20Consultant&location=Germany&f_TPR=r86400"),
        ("LinkedIn - Netherlands (ERP / App Consultant - Past 24h)", 
         "https://www.linkedin.com/jobs/search/?keywords=ERP%20Implementation%20OR%20Application%20Consultant&location=Netherlands&f_TPR=r86400"),
        ("LinkedIn - Ireland (ERP / App Consultant - Past 24h)", 
         "https://www.linkedin.com/jobs/search/?keywords=ERP%20Implementation%20OR%20Application%20Consultant&location=Ireland&f_TPR=r86400"),
        ("StepStone.de - Germany (ERP Implementation Lead - English)", 
         "https://www.stepstone.de/jobs/erp-implementation-lead/in-germany?ag=age_1"),
        ("IamExpat Jobs - Netherlands (IT & ERP)", 
         "https://www.iamexpat.nl/career/jobs-netherlands/it-technology")
    ]
    return links


def send_email(jobs, quick_links):
    today = datetime.now().strftime("%Y-%m-%d")
    msg = MIMEMultipart("alternative")
    msg["Subject"] = f"Daily Europe ERP & Application Jobs ({today}) - {len(jobs)} New Matches"
    msg["From"] = SENDER_EMAIL
    msg["To"] = RECEIVER_EMAIL

    html = f"""
    <html>
    <body style="font-family: Arial, sans-serif; color: #333; line-height: 1.6;">
        <h2 style="color: #1a5276;">Daily European Job Digest ({today})</h2>
        <p>Target Roles: <b>ERP Implementation Lead | Senior Application Consultant | Application Manager</b></p>
        
        <h3 style="background-color: #eaf2f8; padding: 8px;">1. 1-Click Fresh Searches (Posted in Last 24 Hours)</h3>
        <ul>
    """
    for label, link in quick_links:
        html += f'<li><a href="{link}" style="color: #1f618d; font-weight: bold;">{label}</a></li>'
    html += "</ul>"

    html += f'<h3 style="background-color: #eaf2f8; padding: 8px;">2. Automated API Matches ({len(jobs)} Found)</h3>'
    if jobs:
        html += """
        <table border="1" cellpadding="8" cellspacing="0" style="border-collapse: collapse; width: 100%; font-size: 14px;">
            <tr style="background-color: #2c3e50; color: white;">
                <th>Role Title</th>
                <th>Company</th>
                <th>Location</th>
                <th>Note</th>
                <th>Apply Link</th>
            </tr>
        """
        for j in jobs:
            html += f"""
            <tr>
                <td><b>{j['title']}</b></td>
                <td>{j['company']}</td>
                <td>{j['location']}</td>
                <td>{j['visa_tag']}</td>
                <td><a href="{j['url']}" style="background-color: #27ae60; color: white; padding: 5px 10px; text-decoration: none; border-radius: 4px;">Apply</a></td>
            </tr>
            """
        html += "</table>"
    else:
        html += "<p>No new API-indexed ERP matches in this run. Use the 1-Click LinkedIn/StepStone 24h links above.</p>"

    html += """
        <br><p style="font-size: 12px; color: #777;">Automated Job Finder Script | Europe Relocation Pipeline</p>
    </body>
    </html>
    """

    msg.attach(MIMEText(html, "html"))

    try:
        with smtplib.SMTP_SSL("smtp.gmail.com", 465) as server:
            server.login(SENDER_EMAIL, APP_PASSWORD)
            server.sendmail(SENDER_EMAIL, RECEIVER_EMAIL, msg.as_string())
        print("Daily job digest email sent successfully!")
    except Exception as e:
        print(f"Failed to send email: {e}")


if __name__ == "__main__":
    print("Scanning European job boards...")
    all_jobs = fetch_arbeitnow_jobs() + fetch_adzuna_jobs()
    # Deduplicate by URL
    unique_jobs = list({j["url"]: j for j in all_jobs}.values())
    links = build_direct_search_links()
    send_email(unique_jobs, links)

---
name: job-application-resume
description: Tailors a resume to one job end to end - ATS match score and missing keywords, XYZ-formula experience rewrite, new resume (DOCX + PDF) and cover letter, repeated ATS passes until clean, then the 3 hardest interview questions with model answers. Use when the user types /job-application-resume or asks to tailor a resume to a job.
---

# Job Application Resume

Run the stages below **in order**, showing the user the output of each stage before moving on. Do not skip stages. The quoted prompts are the user's own wording; run each one exactly as the task for that stage.

**Hard rules (apply to every stage)**
- Never invent employers, titles, dates, degrees, tools, or numbers. If a metric or skill is missing, ask the user. If they can't supply one, use a truthful non-numeric measure and list it under "Please confirm" at the end.
- Never edit the user's original uploads. Work on copies.
- The resume DOCX and PDF must always contain identical text; regenerate the PDF from the DOCX every time the DOCX changes.
- File names follow `<Full Name>_<Company Name>_<Job Title>` for both .docx and .pdf:
  - Resume: `<Full Name>_<Company Name>_<Job Title>.docx` / `.pdf`
  - Cover letter: `<Full Name>_<Company Name>_<Job Title>_Cover_Letter.docx` / `.pdf` (the suffix stops it overwriting the resume)
  - Take the full name from the resume, and the company and exact job title from the job description. Replace spaces with underscores and drop characters that break file names (`/ \ : * ? " < > | , &`). Example: `Vishal_Raj_Google_Senior_Data_Analyst.pdf`.
  - Use this naming for every version you save or send, including after each ATS fix.

---

## Stage 0 - Collect the three inputs

As soon as the skill is invoked, ask for all three in **one** message, then stop and wait:

1. **Resume** - .docx preferred (.pdf also accepted)
2. **Job description** - PDF, DOCX, screenshot/image, or pasted text
3. **Job posting URL** - reply "none" if there isn't one

If any of these were already given alongside the invocation, don't ask for them again. Once you have them:
- DOCX: extract text with `pandoc file.docx -t plain` or python-docx.
- PDF: `pdftotext -layout file.pdf -` (fall back to pdfplumber; if no text, it's a scan - view the pages as images).
- Screenshot/image: view it and transcribe the full text.
- Save the job description text to `jd.md` and the resume text to `resume_original.txt`.

## Stage 1 - Complete the job description

Check `jd.md` for each field: job title, company, location / remote, seniority / years of experience, must-have skills, nice-to-have skills, responsibilities, tools / tech stack, education / certifications, domain / industry, team or reporting line, salary (if stated).

For any missing field, fetch the URL with WebFetch. If the page is blocked (login wall - common for LinkedIn, Workday, Naukri), try WebSearch for `"<company>" "<job title>"` to find the same posting elsewhere. Still unavailable → mark the field "not stated" and continue. Never guess a field.

Tell the user in 2-3 lines what was filled in from the URL and what is still missing. Append fetched details to `jd.md` with their source.

## Stage 2 - Recruiter analysis

Run this prompt against the resume and completed JD:

> "Act as a senior recruiter with 10+ years of experience. Analyse my resume against this job description. Give me a match score out of 100 and list the top 5 missing keywords that I need to include to pass ATS filters"

How to score (keep it consistent and explainable):

| Area | Weight | What to judge |
|---|---|---|
| Hard skills & tools | 35 | JD must-haves present in the resume (exact term or clear synonym) |
| Experience relevance & seniority | 25 | Years, level, similar scope and domain |
| Responsibilities & impact | 20 | Resume shows the JD's core duties with outcomes |
| Education & certifications | 10 | Required / preferred credentials |
| ATS keyword phrasing | 10 | JD's exact wording used (e.g. "stakeholder management", not only "worked with stakeholders") |

Extract JD keywords first (hard skills, tools, certifications, methods, title terms, key soft skills), then rank missing ones by how often and how prominently the JD uses them (must-have > nice-to-have).

**Respond in exactly this format:**

**Match score: NN/100**

| Area | Score | Notes |
|---|---|---|
(one row per area above)

**Fit:** Strong / Moderate / Weak - 2-3 sentence verdict on whether to apply and the biggest gap.

**Top 5 missing keywords**
1. **Keyword** - where the JD asks for it · why it matters · where it could go in the resume (or ⚠️ "only add if you have this experience")
(…5)

**Other feedback**
- Strengths to keep
- Red flags (gaps, title mismatch, length, weak bullets, no metrics)
- Quick wins

Save as `analysis.md` and the 5 keywords (one per line) as `keywords.txt`.

## Stage 3 - Rewrite the experience section (XYZ)

Take the Stage 2 feedback as input and run:

> "Rewrite my experience section to naturally include those keywords. Use the Google XYZ formula for every bullet point: "Accomplished [X] as measured by [Y] by doing [Z]""

Rules:
- Every bullet has all three parts: X = the result, Y = the measure (number, %, time, money, scale), Z = what the user did. Wording can flow naturally (e.g. "Cut report turnaround 60% (5 days → 2) by automating Excel workflows in Python") as long as X, Y and Z are all there.
- Work the 5 missing keywords in naturally - each at least once in Experience and also in Skills - but only where the user really has that experience.
- Start bullets with strong past-tense verbs; 3-6 bullets per recent role, 2-3 for older roles; one line each where possible, never more than two.
- Also tighten the Summary (3 lines, mirrors the JD title and top keywords) and the Skills section (grouped, JD terms first).

Show the rewritten Experience section, then ask the user everything you need in **one** batched message, for example:
- Real numbers for each bullet whose Y is missing (list those bullets)
- Whether they have hands-on experience with any ⚠️ keyword
- Preferences: 1 or 2 pages, target title line, anything to drop or add, contact details to update
- Hiring manager's name for the cover letter, if known

Wait for answers. If the user says "use your judgment" or doesn't know a number, use a truthful qualitative measure and track it for "Please confirm".

## Stage 4 - Generate the resume (DOCX + PDF) and cover letter

If a `docx` skill is available, read its SKILL.md first. Otherwise build with python-docx (`pip install python-docx`). Build ATS-safe from the start:

- One column. No tables, text boxes, columns, images, icons, skill bars or graphics.
- Name on line 1; email | phone | city | LinkedIn URL on line 2, in the **body** (never in the header/footer).
- Headings exactly: Summary, Skills, Experience, Education, Certifications, Projects (only those that apply).
- One font throughout (Calibri or Arial): body 10.5-11pt, headings 12pt bold, name 16-18pt. Margins 0.5-1 in.
- Each role: `Job Title, Company - City` on one line and `Mon YYYY – Mon YYYY` (or "Present"); one date format everywhere.
- Bullets: Word list bullets or a plain "•" only.
- Length: 1 page if under ~8 years of experience, otherwise max 2.

Convert to PDF from the DOCX: `soffice --headless --convert-to pdf <Full Name>_<Company Name>_<Job Title>.docx` (or `docx2pdf` if LibreOffice isn't installed). Render page 1 of the PDF to an image and look at it to catch layout problems.

**Cover letter** (DOCX + PDF, same font and contact block as the resume): 250-400 words, addressed to the hiring manager by name if known, otherwise "Dear Hiring Manager".
1. Opening - the exact role title and company, and one specific reason this role fits (from the JD / URL, not generic).
2. Two proof paragraphs - each maps a top JD requirement to a real achievement with its metric.
3. Why this company - only facts from the JD or the posting page.
4. Close - confident, with a call to an interview.
No tables, no header/footer, plain paragraphs.

Send all four files to the user.

## Stage 5 - ATS compatibility loop

Take the latest resume (DOCX + PDF) and cover letter as input and run:

> "Act as an applicant tracking system filter (all type). Scan my latest new resume. Tell me which sections a bot would struggle to read which formatting might cause issues, and what I need to fix to ensure 100% ATS compatibility."

Do this as a **real scan**, not an opinion:
1. Save the script below as `ats_check.py` and run it on the resume and on the cover letter:
   `python3 ats_check.py <resume>.docx <resume>.pdf keywords.txt` and `python3 ats_check.py <cover letter>.docx <cover letter>.pdf`
2. Read the parsed text (`pdftotext Resume.pdf -`) yourself as a bot would: is the reading order right, are sections recognisable, does each job show title, company and dates together, are the keywords spelled as in the JD?
3. Apply the common rules of the major ATS families - Workday, Taleo (Oracle), iCIMS, Greenhouse, Lever, SuccessFactors, SmartRecruiters - (tables, headers/footers, columns, graphics, odd headings, mixed dates, symbols, file type and name).

Report each pass as:

**ATS pass N - X issues found**

| # | Issue | Section | Why a bot struggles | Fix |
|---|---|---|---|---|

Then **fix every issue** in the resume DOCX, regenerate the resume PDF, apply the same fixes to the cover letter DOCX/PDF, and run the same prompt again as pass N+1.

- Stop as soon as a pass finds **0 issues** - if pass 1 is already clean, say so and move on.
- Safety cap: stop after 5 passes. Anything still open at that point needs a decision from the user (e.g. a keyword they don't have); list it and ask.
- Send the updated files after the final clean pass (not after every pass).

```python
"""ATS compatibility scan for a resume or cover letter.
Usage: python3 ats_check.py RESUME.docx RESUME.pdf [keywords.txt]
keywords.txt = one JD keyword per line (optional). Prints JSON issues + extracted text stats.
"""
import sys, re, zipfile, subprocess, json
from docx import Document

STD_HEADINGS = {"summary", "professional summary", "profile", "skills", "technical skills", "core skills",
                "experience", "work experience", "professional experience", "education", "certifications",
                "projects", "awards", "publications", "languages", "volunteer experience"}
SAFE_FONTS = {"arial", "calibri", "cambria", "garamond", "georgia", "helvetica", "times new roman",
              "verdana", "tahoma", "aptos", "liberation sans", "liberation serif", "carlito"}
ODD_CHARS = re.compile(r"[\u2605\u2606\u25ba\u25b6\u2713\u2714\u2756\u27a2\u2794\u25cf\u25a0\u25aa\ufb01\ufb02\ufffd\uf0b7]")
issues = []

def add(issue, where, impact, fix):
    issues.append({"issue": issue, "section": where, "ats_impact": impact, "fix": fix})

def check_docx(path):
    z = zipfile.ZipFile(path)
    xml = z.read("word/document.xml").decode("utf8")
    if re.search(r"<w:tbl[ >]", xml):
        add("Tables used for layout", "Body", "Taleo/iCIMS/older parsers read cells out of order or drop them", "Replace tables with plain paragraphs")
    if "w:txbxContent" in xml:
        add("Text boxes", "Body", "Most ATS skip text-box content entirely", "Move text into normal paragraphs")
    if re.search(r'<w:cols [^>]*w:num="[2-9]"', xml):
        add("Multi-column section", "Layout", "Columns get merged line-by-line into garbled text", "Use a single column")
    if "<w:drawing" in xml or "<w:pict" in xml:
        add("Images, icons or shapes", "Layout", "Ignored by ATS; any text inside them is lost", "Remove graphics; use plain text")
    for name in z.namelist():
        if re.match(r"word/(header|footer)\d*\.xml", name):
            t = re.sub(r"<[^>]+>", " ", z.read(name).decode("utf8"))
            if re.search(r"@|\d{3}[\s.-]?\d{3}[\s.-]?\d{4}|linkedin", t, re.I):
                add("Contact details in header/footer", "Header", "Workday/Taleo often skip headers, so name/email go missing", "Put contact details in the body's first lines")
    fonts = {f.lower() for f in re.findall(r'w:ascii="([^"]+)"', xml)}
    bad = fonts - SAFE_FONTS
    if bad:
        add(f"Non-standard fonts: {', '.join(sorted(bad))}", "Typography", "May substitute or mis-encode characters", "Use Calibri, Arial or Garamond")
    doc = Document(path)
    paras = [p for p in doc.paragraphs if p.text.strip()][1:]  # first line is the candidate's name
    heads = [p.text.strip() for p in paras
             if len(p.text.split()) <= 4 and (p.style.name.startswith("Heading") or p.text.isupper())]
    odd = [h for h in heads if h.lower() not in STD_HEADINGS and not re.search(r"@|\d", h)]
    if odd:
        add(f"Possibly non-standard headings: {odd}", "Headings", "ATS map sections by known heading names", "Rename to Summary / Skills / Experience / Education / Certifications / Projects")
    if ODD_CHARS.search(xml):
        add("Decorative symbols or ligatures", "Body", "Render as junk characters in parsed text", "Use plain • bullets or Word list bullets only")

def check_pdf(path, keywords):
    txt = subprocess.run(["pdftotext", path, "-"], capture_output=True, text=True).stdout
    if len(txt.strip()) < 200:
        add("PDF text not extractable", "Whole file", "Image-only PDF reads as blank", "Export the PDF from the DOCX, never print-to-image")
        return txt
    top = "\n".join(txt.strip().splitlines()[:12])
    if not re.search(r"[\w.+-]+@[\w-]+\.[\w.]+", top):
        add("Email not found near the top of parsed text", "Contact", "Candidate record created without email", "Put email in the first lines of the body")
    if not re.search(r"\+?\d[\d\s().-]{8,}\d", top):
        add("Phone not found near the top of parsed text", "Contact", "Recruiter can't call from the ATS record", "Put phone in the first lines of the body")
    if ODD_CHARS.search(txt):
        add("Junk characters in parsed PDF text", "Body", "Garbled keywords fail matching", "Remove symbols/ligatures; use standard fonts")
    styles = set()
    for pat, label in [(r"\b(Jan|Feb|Mar|Apr|May|Jun|Jul|Aug|Sep|Oct|Nov|Dec)[a-z]*\.? \d{4}", "Mon YYYY"),
                       (r"\b\d{1,2}/\d{4}", "MM/YYYY"), (r"\b\d{4}-\d{2}\b", "YYYY-MM")]:
        if re.search(pat, txt):
            styles.add(label)
    if len(styles) > 1:
        add(f"Mixed date formats: {sorted(styles)}", "Experience", "Tenure calculation can fail", "Use one format everywhere, e.g. 'Jan 2022 – Mar 2024'")
    missing = [k for k in keywords if k and k.lower() not in txt.lower()]
    if missing:
        add(f"JD keywords absent from parsed text: {missing}", "Keywords", "Keyword-match score drops", "Add each where it is truthful (Skills + one Experience bullet)")
    return txt

if __name__ == "__main__":
    kws = [l.strip() for l in open(sys.argv[3])] if len(sys.argv) > 3 else []
    check_docx(sys.argv[1])
    text = check_pdf(sys.argv[2], kws)
    print(json.dumps({"issue_count": len(issues), "issues": issues, "parsed_words": len(text.split())}, indent=2))
```

(Needs `python-docx` and `pdftotext` from poppler-utils.)

## Stage 6 - Hiring manager interview prep

Take the final resume, cover letter and the JD as input and run:

> "Act as the hiring manager for this specific role. Ask me the 3 hardest technical questions you would ask in an interview. Then give me the perfect response based on my background and the job description"

- "Technical" means the hardest role-specific questions: system/tool depth for engineering or data roles, domain and case questions for business, finance, marketing or ops roles.
- Pick questions that probe the JD's most important requirement and the candidate's weakest area from Stage 2.

For each question:

**Q1. The question**
- *What the hiring manager is really testing*
- **Perfect answer** (first person, 150-250 words, built only from the user's real experience in the resume; use STAR for experience-based questions and show reasoning for problem-solving ones)
- *Likely follow-up* and one line on how to handle it

## Close

Finish with a short wrap-up and end the skill:
- Final files (resume DOCX + PDF, cover letter DOCX + PDF) - send them if not already sent after the final ATS pass
- Match score before → estimated after, and the keywords added
- Number of ATS passes and the result
- **Please confirm** - any wording that used a qualitative measure instead of a real number

Then say the job application workflow is complete.

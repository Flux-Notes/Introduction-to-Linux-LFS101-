# 🐧 Introduction to Linux — LFS101 Notes

> *"The Linux philosophy is 'Laugh in the face of danger'. Oops. Wrong one. 'Do it yourself'. Yes, that's it."*
> — **Linus Torvalds**

A structured, beginner-friendly documentation for **Introduction to Linux (LFS101)** — built with [MkDocs Material](https://squidfunk.github.io/mkdocs-material/) and hosted on GitHub Pages.

![MkDocs](https://img.shields.io/badge/MkDocs-Material-blue?logo=materialformkdocs)
![GitHub Pages](https://img.shields.io/badge/Hosted-GitHub%20Pages-222?logo=github)
![License](https://img.shields.io/badge/License-MIT-green)
![Contributions Welcome](https://img.shields.io/badge/Contributions-Welcome-orange)
![Linux](https://img.shields.io/badge/Linux-LFS101-FCC624?logo=linux&logoColor=black)

---

## 📚 Topics Covered

<details>
<summary>🏛️ Foundations</summary>

- The Linux Foundation
- Linux Philosophy and Concepts

</details>

<details>
<summary>🚀 Getting Started</summary>

- Linux Basics and System Startup
- Graphical Interface
- System Configuration from Graphical Interface
- Common Applications

</details>

<details>
<summary>💻 Command Line Operations</summary>

- Command Line Operations
- Finding Linux Documentation
- Text Editors

</details>

<details>
<summary>⚙️ System Management</summary>

- Processes
- File Operations
- User Environment
- Manipulating Text

</details>

<details>
<summary>🌐 Networking & Scripting</summary>

- Network Operations
- Bash Shell and Basic Scripting
- More on Bash Shell Scripting

</details>

<details>
<summary>🔒 Advanced Topics</summary>

- Printing
- Local Security Principles

</details>

---

## 🛠️ Built With

| Tool | Purpose |
|------|---------|
| [MkDocs](https://www.mkdocs.org/) | Static site generator |
| [Material for MkDocs](https://squidfunk.github.io/mkdocs-material/) | Theme |
| [GitHub Pages](https://pages.github.com/) | Hosting |
| Markdown | Content writing |

---

## 🚀 Run Locally

Follow these steps to run the project on your machine:

### 1. Clone the repo
```bash
git clone https://github.com/Flux-Notes/Introduction-to-Linux-LFS101-.git
cd Introduction-to-Linux-LFS101-
```

### 2. Create & activate virtual environment
```bash
python -m venv venv

# Mac/Linux
source venv/bin/activate

# Windows
venv\Scripts\activate
```

### 3. Install dependencies
```bash
pip install mkdocs mkdocs-material
```

### 4. Preview locally
```bash
mkdocs serve
```
Open [http://127.0.0.1:8000](http://127.0.0.1:8000) in your browser.

### 5. Deploy to GitHub Pages
```bash
mkdocs gh-deploy
```

---

## 📁 Project Structure

```
Intro-to-Linux/
├── docs/
│   ├── .nojekyll
│   ├── index.md
│   ├── 01-the-linux-foundation.md
│   ├── 02-linux-philosophy-and-concepts.md
│   ├── 03-linux-basics-and-system-startup.md
│   ├── 04-graphical-interface.md
│   ├── 05-system-configuration-from-graphical.md
│   ├── 06-common-applications.md
│   ├── 07-command-line-operations.md
│   ├── 08-finding-linux-documentation.md
│   ├── 09-processes.md
│   ├── 10-file-operations.md
│   ├── 11-text-editors.md
│   ├── 12-user-environment.md
│   ├── 13-manipulating-text.md
│   ├── 14-network-operations.md
│   ├── 15-bash-shell-and-basic-scripting.md
│   ├── 16-more-on-bash-shell-scripting.md
│   ├── 17-printing.md
│   └── 18-local-security-principles.md
├── mkdocs.yml
├── .gitignore
└── README.md
```

---

## 🤝 Contributing

Contributions are welcome and appreciated! Whether it's fixing a typo, improving an explanation, or adding new examples — every bit helps. 🙌

### ✅ What You Can Contribute

- Fix typos or grammar mistakes
- Improve or simplify existing explanations
- Add better code examples or diagrams
- Add missing topics or subtopics
- Fix broken links or formatting issues

---

### 📋 Contribution Steps

**1. Fork the repository**

Click the **Fork** button at the top right of this page.

**2. Clone your fork**
```bash
git clone https://github.com/Flux-Notes/Introduction-to-Linux-LFS101-.git
cd Introduction-to-Linux-LFS101-
```

**3. Create a new branch**
```bash
git checkout -b your-branch-name
# Example: git checkout -b fix/processes-typo
# Example: git checkout -b add/bash-scripting-examples
```

**4. Set up the project locally**
```bash
python -m venv venv
source venv/bin/activate     # Mac/Linux
venv\Scripts\activate        # Windows

pip install mkdocs mkdocs-material
```

**5. Make your changes**

Edit the relevant `.md` file inside the `docs/` folder.

**6. Preview your changes**
```bash
mkdocs serve
```
Open [http://127.0.0.1:8000](http://127.0.0.1:8000) and verify everything looks correct.

**7. Commit your changes**
```bash
git add .
git commit -m "fix: corrected typo in 09-processes.md"
```

**8. Push to your fork**
```bash
git push origin your-branch-name
```

**9. Open a Pull Request**

Go to the original repo on GitHub and click **"New Pull Request"**. Describe what you changed and why.

---

### 📝 Commit Message Format

Please follow this format for commit messages:

| Prefix | When to use |
|--------|-------------|
| `fix:` | Bug fix, typo correction |
| `add:` | New content or topic added |
| `update:` | Improving existing content |
| `remove:` | Removing outdated content |
| `style:` | Formatting changes only |

**Examples:**
```
fix: corrected command example in 07-command-line-operations.md
add: added examples in 15-bash-shell-and-basic-scripting.md
update: improved explanation in 14-network-operations.md
style: fixed code block formatting in 13-manipulating-text.md
```

---

### 🗂️ Writing Style Guide

To keep the docs consistent, please follow these guidelines when writing content:

- Use **simple, beginner-friendly language**
- Always include a **code example** or diagram where applicable
- Add **expected output** after every command block
- Use admonitions for tips, warnings, and notes:
  ```
  !!! tip "Title"
      Your tip here.

  !!! warning "Title"
      Your warning here.

  !!! info "Title"
      Your info here.
  ```
- Use **tables** for comparisons (e.g., `bash` vs `sh`)
- Keep examples **short and focused**

---

### 🐛 Reporting Issues

Found a mistake or something that can be improved?

1. Go to the [Issues](https://github.com/Flux-Notes/Introduction-to-Linux-LFS101-/issues) tab
2. Click **"New Issue"**
3. Describe the problem clearly with the page name and what needs to be fixed

---

## 📄 License

This project is open source and available under the [MIT License](LICENSE).

---

⭐ If you find this helpful, give it a star — it keeps the motivation going!
# Product Features: Learning With Texts (LWT)

This document describes the main features of the LWT application, how they work, and their purpose.

---

## 1. Language Management
- **Define and manage multiple languages** you want to learn.
- **Customize sentence and word splitting** for each language.
- **Set up and manage web dictionaries** for quick lookups.

**How it works:**
- Users can add new languages, configure how text is parsed, and link to online dictionaries for instant reference during reading and testing.

---

## 2. Text Management
- **Upload and organize texts** for study.
- **Automatic splitting** of texts into sentences and words.
- **Associate audio files** (optional) with texts for listening practice.
- **Tag texts** for better organization.

**How it works:**
- Users import texts (with optional audio), which are then parsed into manageable units. Texts can be tagged and organized for easy retrieval.

---

## 3. Vocabulary (Term) Management
- **Click on words or expressions** in texts to look up meanings and save them as "terms" (words or multi-word expressions).
- **Save translations, romanizations, and example sentences** for each term.
- **Edit, update, and manage the status** of terms (unknown, learning, learned, well-known, ignored).
- **Import and export terms** in CSV/TSV format.
- **Export terms for use in Anki** (flashcard software), including cloze deletion support.

**How it works:**
- While reading, users can click on any word or phrase to add it to their vocabulary list, annotate it, and track its learning status. Terms can be imported/exported for use in other tools.

---

## 4. Learning and Testing
- **Read texts with visual cues** for known/unknown words.
- **Test your understanding** of words and expressions, both in and out of sentence context.
- **Built-in support for MCD (Massive-Context Cloze Deletion) testing.**
- **Track your progress** with a statistics page.

**How it works:**
- The system highlights words based on their learning status. Users can initiate tests (recognition, recall, cloze, or table-based) to reinforce vocabulary. Progress is tracked and visualized.

---

## 5. Database and Backup
- **Backup and restore your database.**
- **Support for multiple "table sets"** (multi-user or multi-language environments within one database).

**How it works:**
- Users can create backups or restore their data, and advanced users can manage multiple sets of data for different languages or users.

---

## 6. User Interface
- **Web-based interface** accessible from modern browsers.
- **Experimental mobile version** for learning on the go.
- **Keyboard shortcuts** for efficient navigation and learning.

**How it works:**
- The application is accessed via a web browser, with a responsive design and keyboard shortcuts to speed up common actions.

---

## 7. Integration and Extensibility
- **Open source and public domain:** free to use, modify, and extend.
- **Can be integrated with WordPress** for single sign-on.
- **Portable and easy to deploy** (especially with Docker).

**How it works:**
- The codebase is open and can be customized or extended. Integration with other platforms (like WordPress) is possible for advanced setups.

---

## Summary Table of Key Features

| Feature Area         | What You Can Do                                      | How It Works / Purpose                        |
|---------------------|------------------------------------------------------|-----------------------------------------------|
| Language Management | Add languages, set parsing, link dictionaries        | Customizes learning for each language         |
| Text Management     | Import texts/audio, organize, tag                    | Provides learning material and context        |
| Vocabulary          | Save/edit terms, import/export, Anki integration     | Builds and manages your personal vocabulary   |
| Learning & Testing  | Visual cues, multiple test modes, stats              | Reinforces learning and tracks progress       |
| Database/Backup     | Backup/restore, multi-table support                  | Data safety and multi-user/multi-language use |
| User Interface      | Web/mobile UI, keyboard shortcuts                    | Easy, efficient access from any device        |
| Integration         | Open source, WordPress SSO, Dockerized deployment    | Extensible and easy to set up                 |

---

For more details on each feature or technical implementation, see the main documentation or contact the project maintainers. 
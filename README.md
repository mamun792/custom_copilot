# custom_copilot

Repository for GitHub Copilot instructions and a full-stack engineering reference.

## Purpose
- Keep a compact Copilot instruction set for daily use.
- Maintain a comprehensive reference document for architecture, security, and system design.

## Contents
- Copilot instructions: [.github/copilot-instructions.md](.github/copilot-instructions.md)
- Full reference guide: [.github/dev-reference-EN.md](.github/dev-reference-EN.md)

## Setup
1. Clone the repo.
2. Open in VS Code.
3. Review the files under .github/.

## Run / Deploy
- No runtime or deployment steps are required. This is a docs-only repository.

## Usage
1. Open this repo in VS Code.
2. Use the Copilot instructions file for day-to-day coding guidance.
3. Use the full reference file for architecture, security, and system design details.

## Contributing
1. Create a feature branch.
2. Make changes to the relevant docs.
3. Keep instructions concise and consistent.
4. Open a PR with a clear description of what changed and why.

## Notes
- Keep instructions concise in the Copilot file.
- Use the reference file for deep dives and upload to LLMs when needed.
১) VS Code‑এ রিপো খুলুন

রিপো ওপেন করুন এবং Copilot Chat চালু রাখুন।
২) Copilot instructions ফাইল ব্যবহার

প্রতিদিনের কাজের জন্য সংক্ষিপ্ত নির্দেশনা আছে copilot-instructions.md।
কোড লেখার সময় Copilot এই ফাইলের নিয়ম অনুসরণ করবে।
৩) Full reference ফাইল ব্যবহার

বড় আর্কিটেকচার/সিকিউরিটি/ডিজাইন রেফারেন্স আছে dev-reference-EN.md।
বড় সিদ্ধান্ত/পলিসি/ডিটেইল দেখতে এই ফাইল খুলে রেফার করুন।
৪) Workspace‑এ Copilot instructions সেট করা (optional কিন্তু ভালো)

VS Code‑এ .vscode/settings.json এ নিচের মতো যোগ করতে পারেন:

{  "github.copilot.chat.codeGeneration.instructions": [    { "file": ".github/copilot-instructions.md" }  ]}
এতে সব ফাইলে Copilot একই নির্দেশনা মানবে।
৫) ব্যবহারযোগ্য প্রম্পট উদাহরণ

“PlaceOrderCommand + Handler বানাও CQRS pattern অনুযায়ী; Repository pattern এবং Domain event যোগ করো।”
“Payment integration যোগ করো Strategy pattern‑এ, webhook signature verify + idempotency দাও।”
৬) নতুন প্রজেক্টে কাজ শুরু করলে

SRS/ফিচার লিস্ট, ERD, আর্কিটেকচার ডায়াগ্রাম দিন।
আমি তারপর আর্কিটেকচার ডিসিশন, ফোল্ডার স্ট্রাকচার, মাইগ্রেশন, ফেজ প্ল্যান করে দেব।
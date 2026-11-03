# คู่มือติดตั้ง Thai Codex Skills

คู่มือนี้อธิบายวิธีนำสกิลจาก repository นี้ไปติดตั้งใน Codex โดยรองรับทั้งการติดตั้งครบทุกสกิลและการติดตั้งทีละสกิล

Repository: https://github.com/thakot-cloud/SKILL-Me

## โครงสร้างของ repository

สกิลถูกแยกเป็นหมวดหมู่ แต่ละสกิลจะมีไฟล์ SKILL.md อยู่ที่โฟลเดอร์ระดับสกิล:

skills/<หมวด>/<ชื่อสกิล>/SKILL.md

ปัจจุบันมี 84 สกิลใน 11 หมวด

## วิธีติดตั้งทั้งหมดบน Windows

เปิด PowerShell แล้วรันคำสั่งต่อไปนี้:

~~~powershell
$repo = Join-Path $env:USERPROFILE 'SKILL-Me'
$dest = Join-Path $env:USERPROFILE '.codex\skills'

if (Test-Path -LiteralPath $repo) {
    git -C $repo pull --ff-only
} else {
    git clone https://github.com/thakot-cloud/SKILL-Me.git $repo
}

New-Item -ItemType Directory -Path $dest -Force | Out-Null

$skillDirs = @(
    Get-ChildItem -LiteralPath (Join-Path $repo 'skills') -Directory -Recurse |
    Where-Object { Test-Path -LiteralPath (Join-Path $_.FullName 'SKILL.md') }
)

foreach ($skillDir in $skillDirs) {
    Copy-Item -LiteralPath $skillDir.FullName -Destination $dest -Recurse -Force
}

Write-Host ("ติดตั้งแล้ว {0} สกิล" -f $skillDirs.Count)
~~~

คำสั่งนี้จะคัดลอกเฉพาะโฟลเดอร์ที่มี SKILL.md ไปไว้ที่ C:\Users\<ชื่อผู้ใช้>\.codex\skills และจะอัปเดตไฟล์เดิมให้ด้วยเมื่อรันซ้ำ

## วิธีติดตั้งทั้งหมดบน macOS หรือ Linux

เปิด Terminal แล้วรัน:

~~~bash
repo="$HOME/SKILL-Me"
if [ -n "$CODEX_HOME" ]; then
  dest="$CODEX_HOME/skills"
else
  dest="$HOME/.codex/skills"
fi

if [ -d "$repo/.git" ]; then
  git -C "$repo" pull --ff-only
else
  git clone https://github.com/thakot-cloud/SKILL-Me.git "$repo"
fi

mkdir -p "$dest"

while IFS= read -r -d '' skill_file; do
  skill_dir="$(dirname "$skill_file")"
  cp -R "$skill_dir" "$dest/"
done < <(find "$repo/skills" -mindepth 2 -type f -name SKILL.md -print0)
~~~

หากกำหนดตัวแปร CODEX_HOME ไว้ ระบบจะใช้โฟลเดอร์นั้นแทน ~/.codex

## ติดตั้งเพียงสกิลเดียว

ถ้ามีสกิลติดตั้งอยู่แล้วในเครื่อง ให้ใช้ตัวติดตั้งของ Codex โดยระบุ path ของโฟลเดอร์สกิล เช่น:

~~~powershell
python "$env:USERPROFILE\.codex\skills\.system\skill-installer\scripts\install-skill-from-github.py" --repo thakot-cloud/SKILL-Me --path skills/content/caption-writer-th --dest "$env:USERPROFILE\.codex\skills"
~~~

เปลี่ยน --path เป็น path ของสกิลที่ต้องการ เช่น:

~~~text
skills/documents/thai-pdf
skills/thai-language/thai-address
skills/marketing/seo-audit-th
~~~

ตัวติดตั้งจะหยุดถ้าโฟลเดอร์ชื่อเดียวกันมีอยู่แล้ว หากต้องการอัปเดตสกิลที่ติดตั้งไว้ ให้ใช้วิธีติดตั้งทั้งหมดด้านบนหรือคัดลอกไฟล์ทับตามขั้นตอนอัปเดต

## ตรวจสอบว่าติดตั้งสำเร็จ

บน Windows:

~~~powershell
$dest = Join-Path $env:USERPROFILE '.codex\skills'
@(Get-ChildItem -LiteralPath $dest -Filter 'SKILL.md' -File -Recurse).Count
~~~

บน macOS หรือ Linux:

~~~bash
if [ -n "$CODEX_HOME" ]; then
  find "$CODEX_HOME/skills" -type f -name SKILL.md | wc -l
else
  find "$HOME/.codex/skills" -type f -name SKILL.md | wc -l
fi
~~~

จำนวนที่ได้อาจมากกว่า 84 หากมีสกิลอื่นติดตั้งอยู่ก่อนแล้ว ให้ตรวจสอบเพิ่มเติมว่าสกิลจากชุดนี้มีโฟลเดอร์อยู่ในปลายทาง เช่น:

~~~text
~/.codex/skills/thai-language/thai-address/SKILL.md
~/.codex/skills/content/caption-writer-th/SKILL.md
~~~

หลังติดตั้ง ให้เริ่มบทสนทนาใหม่หรือรีสตาร์ต Codex เพื่อให้ระบบโหลดรายการสกิลล่าสุด

## อัปเดตสกิลในภายหลัง

เมื่อมีการอัปเดตใน repository ให้รันวิธีติดตั้งทั้งหมดซ้ำได้เลย คำสั่งจะ git pull แล้วคัดลอกสกิลเวอร์ชันล่าสุดไปยังโฟลเดอร์ Codex

หากต้องการเก็บเวอร์ชันเดิมไว้ก่อน ให้สำรองโฟลเดอร์สกิลที่มีชื่อเดียวกันก่อนรันคำสั่ง โดยเฉพาะสกิลที่เคยแก้ไขไฟล์เอง

## ปัญหาที่พบบ่อย

- **Codex ไม่พบสกิล**: ตรวจสอบว่าไฟล์อยู่ในรูปแบบ skills/<ชื่อสกิล>/SKILL.md ที่ปลายทาง ไม่ใช่ skills/<หมวด>/<ชื่อสกิล>/SKILL.md เพราะตอนติดตั้งต้องคัดลอกโฟลเดอร์สกิลด้านในออกมา
- **ติดตั้งไม่ครบ**: ตรวจสอบว่า Git clone สำเร็จ และรันคำสั่งตรวจสอบจำนวน SKILL.md อีกครั้ง
- **สคริปต์ของสกิลทำงานไม่ได้**: สกิลบางรายการมี dependency ของตัวเอง ให้เปิด requirements.txt หรือคำแนะนำใน SKILL.md ของสกิลนั้น
- **ชื่อสกิลซ้ำกับของเดิม**: สำรองโฟลเดอร์เดิมก่อน แล้วค่อยคัดลอกเวอร์ชันจาก repository นี้ทับ

ไฟล์ประกอบและสคริปต์ภายในแต่ละสกิลอยู่ภายใต้ license ของต้นทาง ควรรักษาไฟล์ license เมื่อคัดลอกหรือแจกจ่ายต่อ

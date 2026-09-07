# Yojana Sathi — 10 minute YouTube script

**Language:** Hinglish (Roman script — read it as written)
**Format:** Voiceover + screen recording. No camera.
**Length:** ~1,500 words ≈ 10 minutes at a normal speaking pace.

`[square brackets]` = screen cue. Do not read them.
`⚡` = pattern interrupt — change the visual here, or the viewer drifts.

---

## 0:00 — HOOK (35 words · 14.5s at normal pace)

> Read this at pace, not slowly. It is 14.5 seconds at a normal speaking speed
> and 16 if you drag it. 20–40% of viewers leave inside the first 10 seconds, so
> this is the one place in the video where speed matters more than gravitas.

[screen: black. White text: ₹455]

Ek din ki dihadi — chaar sau pachpan rupaye.

[screen: text changes to "1 galat kaagaz"]

Ek aadmi poora din chhod ke sarkari office jaata hai. Aur wahan se wapas bhej
diya jaata hai — galat kaagaz.

[screen: text — "Toh maine ek app banaya."]

Toh maine ek app banaya.

⚡

---

## 0:15 — WHO I AM (135 words)

[screen: terminal, `neofetch` or your desktop. Then the GitHub profile.]

Hello doston. Main hoon Avinash Negi. Aur aaj main aapko is app ki poori kahani
sunaunga — kya bana, kya toota, aur kya aaj bhi galat hai.

Main Kotdwara se hoon — Uttarakhand ka ek chhota sa town. Yahan koi tech
industry nahi hai. Main online degree kar raha hoon, aur Python backend seekh
raha hoon — mostly khud se, YouTube aur documentation se.

Aur main honest rahunga aapse: abhi tak mera koi job nahi hai. Koi company ka
experience nahi hai. Sirf projects hain.

Toh agar aap bhi wahi ho — degree chal rahi hai, job nahi hai, aur samajh nahi
aa raha ki kya banayein jo actually matter kare — ye video aapke liye hi hai.

Kyunki main aapko koi tutorial nahi dikha raha. Main aapko ek real project ki
poori kahani dikha raha hoon. Achhi bhi, aur buri bhi.

⚡

---

## 1:00 — THE PROBLEM (150 words)

[screen: myScheme.gov.in scrolling on laptop. English text, filters, forms.]

Toh problem kya hai.

India mein sarkar ki bahut saari welfare schemes hain. Insurance hai, pension
hai, registration hai. Ye sab exist karti hain. Paisa allocate ho chuka hai.

Problem information ki hai.

Ye saari jaankari online hai — lekin English mein. Aap padh sakte ho, browser
chala sakte ho, aur aapko pata hona chahiye ki "land holding in hectares" ka
matlab kya hai.

Ab sochiye ek mistri ke baare mein. Ek driver. Ek daily wage labourer. Jise ye
schemes sabse zyada chahiye — uske liye ye teen assumptions bahut zyada hain.

Toh woh kya karta hai? Woh chalke jaata hai. Poora din chhodta hai. Aur agar
jawab "nahi" nikla, ya galat kaagaz nikla — toh chaar sau pachpan rupaye gaye,
aur kiraya alag.

Yahi cost hai. Aur poora project isi ek number ke aas paas bana hai.

⚡

---

## 2:00 — THE PROJECT I THREW AWAY (110 words)

[screen: the Saans repo on GitHub. Scroll it. Then close the tab.]

Ab ek baat jo shayad main chhupa sakta tha, lekin nahi chhupaunga.

Ye mera doosra project hai. Pehla project alag tha — naam tha Saans. Air
pollution ka personal agent. Telegram par. Around saade teen hazaar lines.
Zero dependencies. Sab checks pass. Live bot. Public repo.

Woh finished code hai. Aur maine use 30 August ko park kar diya.

Kyun? Kyunki maine use kabhi deploy nahi kiya tha. Ek bhi real user ke saamne
nahi rakha tha.

Aur pehle maine khud ko ek doosra reason bataya tha — ki idea already exist
karta hai. Lekin sach ye hai, woh ek feeling thi. Zero users hona — woh ek
score hai.

Lesson: deploy pehle karo. Polish baad mein.

⚡

---

## 2:45 — FOUR DECISIONS BEFORE ANY CODE (190 words)

[screen: `sathi/rules/engine.py` open. Then `tests/test_rules.py`.]

Toh Scheme Sathi shuru karne se pehle maine chaar decisions liye. Code likhne
se pehle. Aur inhe kabhi change nahi kiya.

**Ek. Rule engine language model ko import nahi kar sakta.**

Model conversation kar sakta hai. Hindi rephrase kar sakta hai. Lekin woh kabhi
koi threshold nahi dekhega, koi rupaye ka figure nahi banayega, aur koi verdict
nahi dega. Eligibility normal, boring, readable Python code decide karta hai.
Aur ek test check karta hai ki galti se bhi koi model import na ho jaye.

**Do. Do nahi, teen jawab.**

Har criterion ka answer hai — haan, nahi, ya **pata nahi**. Missing information
ko chupchaap "nahi" nahi maana jaata.

Sochiye kyun. Galat "haan" — banda din bhar ka kaam chhod ke jaata hai bekaar
mein. Galat "nahi" — usko scheme milti hi nahi jiska woh haqdaar tha. Aur "mujhe
nahi pata, centre par ye sawaal poochhiye" — iska cost zero hai, aur phir bhi
kaam ka hai.

**Teen. Jo value research nahi hui, woh literally "TODO" likhi hai. Number bhi.**

Zero nahi likhta. Kyunki zero dekh ke lagta hai ki kisi ne check kiya hai.

**Chaar. Naam, phone number, Aadhaar ka field hai hi nahi.**

"Hum store nahi karte" nahi — field exist hi nahi karta. Toh galti se bhi store
nahi ho sakta.

⚡

---

## 4:00 — HOW IT ACTUALLY WORKS (150 words)

[screen: phone recording. WhatsApp. /start → language → consent → questions.]

Ab dekhte hain chalta kaise hai.

Ye ek conversation hai. WhatsApp par, aur Telegram par. Hindi mein ya English
mein.

Ye wahi sawaal poochta hai jo ek clerk poochta — aap kis rajya se ho, umar kya
hai, kaam kya karte ho, kamai kitni hai.

[screen: tapping buttons, then the state question rendering as a list]

Har jawab ek button hai. Kuch type nahi karna. Kuch spelling nahi karni.

[screen: results screen, scroll slowly through the ₹ figures]

Aur end mein teen cheezein batata hai. Kya milega. Saal ka kitne rupaye ka
milega. Aur kahan jaana hai.

Do lakh rupaye ka accident cover — bees rupaye saal mein. Teen hazaar rupaye
mahina pension, saath saal ki umar se.

Aur e-Shram — jo apne aap se kuch nahi deta. Toh app saaf bolta hai ki kuch
nahi milta. Bade dikhne ke liye jhoota number nahi banata.

⚡

---

## 5:00 — RE-HOOK (50 words)

[screen: cut to black. Then red text: "Ab galtiyan."]

Ab tak maine aapko wo dikhaya jo sahi bana.

Ab main aapko wo dikhaunga jo maine galat kiya. Kyunki agar aap sirf achha wala
hissa dekhoge, toh aapko lagega ki ye seedha bana. Aisa bilkul nahi hua.

Pehle ghante mein hi chhe bugs mile.

---

## 5:20 — WHAT BROKE (180 words)

[screen: BUILD_LOG.md, bugs section, scrolling]

Bot live hua. Maine khud usko ek worker ki tarah use kiya. Ek hi session mein
chhe bugs.

Aur dhyan dijiye — **saare ke saare conversation wale hisse mein the. Rule
engine mein ek bhi nahi.**

Jaise, ek khaali keyboard bhejne par Telegram ne chupchaap chaar sau ka error
diya — na kuch crash hua, na kuch log hua. Session bas mar gaya.

Ek jagah aisa loop tha jisse nikalne ka raasta hi nahi tha. Maine pehli baar
mein hi phas gaya, "i dont do any job" type karke.

[screen: the `is_verified` property in the diff]

Lekin sabse bura bug maine dhoonda hi nahi. Maine repo public kiya, aur ek AI
review karwaya. Usne ek cheez pakdi jo sach mein serious thi.

Mere code mein "verified" ka matlab tha — "isme koi TODO nahi bacha". Lekin
mere teeno scheme files poori research ho chuki thi. Toh unme koi TODO tha hi
nahi. Matlab code keh raha tha "verified", jabki kisi insaan ne unhe confirm
kiya hi nahi tha.

Ek property. Ek galat line. Aur poora safety gate bekaar.

⚡

---

## 6:30 — CTA (50 words)

[screen: the GitHub repo, then a subscribe animation]

Agar aapko ye tarah ki cheezein interesting lagti hain — poora code open source
hai, link description mein hai. Build log bhi wahi hai, saari galtiyon ke saath.

Aur agar aap bhi apna pehla real project banane ki koshish kar rahe ho, toh
subscribe kar lijiye. Main yahi sab banata rehta hoon, publicly.

---

## 6:50 — SHIPPING IT (180 words)

[screen: terminal, ssh into the EC2 box. `systemctl status sathi`.]

Ab deployment.

Ye AWS ke ek chhote se server par chalta hai, Mumbai region mein. Mahine ka
kareeb dus dollar.

[screen: `deploy/install-on-vm.sh` scrolling]

Deploy ek script hai. Code copy karta hai, secrets alag rakhta hai, aur service
restart karta hai. Aur — ye important hai — **deploy se pehle saare tests
chalata hai.** Agar test fail ho, deploy hota hi nahi.

Systemd ke through chalta hai, matlab server reboot ho jaye toh bhi apne aap
wapas start ho jaata hai.

[screen: Telegram bot conversation]

Pehle Telegram live hua. Telegram aasan hai — ek token lo, polling shuru karo,
ho gaya. Koi verification nahi, koi business account nahi.

Aur maine deliberately Telegram pehle kiya. Kyunki agar main WhatsApp ka
intezaar karta, toh Meta ka approval mere deploy ko rok sakta tha. Core code
channel-agnostic likha — taaki channel badalna sirf ek file ka kaam ho.

Yahi cheez baad mein bahut kaam aayi.

⚡

---

## 8:00 — WHATSAPP, AUR DO BUGS (190 words)

[screen: Meta developer dashboard, WhatsApp configuration page]

Ab WhatsApp.

WhatsApp Cloud API Telegram se kaafi zyada complicated hai. App banao, business
portfolio banao, phone number ID lo, app secret lo, webhook lagao, HTTPS chahiye
— plain HTTP se kaam nahi chalega.

Maine sab set kiya. Dashboard ne bola sab theek hai. Green tick.

Aur phir — kuch nahi aaya. Ek bhi message webhook tak nahi pahuncha.

[screen: the `subscribed_apps` API response]

Do ghante lage. Reason ye tha: dashboard par jo "messages — Subscribed" toggle
maine on kiya tha, usne mere app ko subscribe kiya hi nahi tha. Usne Meta ka
apna demo app subscribe kar rakha tha.

Ek API call se pata chala. Ek aur API call se fix hua. Dashboard jhooth bol raha
tha.

[screen: the sqlite error in the journal]

Doosra bug aur maza aaya. WhatsApp chal gaya — lekin har message par bot bolta
"kuch gadbad ho gayi, dobara /start bhejein".

Reason: mera database connection ek thread mein bana tha, aur WhatsApp doosre
thread se likh raha tha. SQLite ye allow nahi karta.

Telegram mein ye bug kabhi dikha hi nahi — kyunki Telegram single thread par
chalta hai. **Do saal purana code, aur bug tabhi dikha jab doosra channel aaya.**

⚡

---

## 9:15 — WHAT'S STILL WRONG, AND CLOSE (140 words)

[screen: `docs/BUILD_LOG.md` — "What is still open, and what is still wrong"]

Ab ek aakhri baat. Aur ye woh hissa hai jo log videos mein nahi dikhate.

Ye project abhi bhi complete nahi hai.

Teeno schemes par abhi tak kisi insaan ne sign off nahi kiya hai. Matlab aaj ke
din, app har worker ko "mujhe nahi pata" bolta hai — har scheme ke liye. Jaan
boojh ke. Kyunki jab tak koi apna naam nahi likhta us number ke saath, engine
verdict dene se mana kar deta hai.

Aur ek open sawaal hai — e-Shram ki umar ki limit. Ek official page kehta hai
"solah se upar", doosra kehta hai "solah se unsaath". Agar doosra sahi hai, toh
aaj saath saal se upar wale har aadmi ko galat jawab mil raha hai.

Ye main chhupa sakta tha. Lekin poora project isi cheez ke liye bana hai — ki
jab pata na ho, toh bolo ki pata nahi.

[screen: end card]

Main Avinash Negi. Code neeche link mein hai. Milte hain agle video mein.

---

# PRODUCTION NOTES

## Word count by segment

| Segment | Words | Running |
|---|---|---|
| Hook | 40 | 0:15 |
| Who I am | 120 | 1:00 |
| Problem | 150 | 2:00 |
| Parked project | 110 | 2:45 |
| Four decisions | 190 | 4:00 |
| How it works | 150 | 5:00 |
| Re-hook | 50 | 5:20 |
| What broke | 180 | 6:30 |
| CTA | 50 | 6:50 |
| Shipping | 180 | 8:00 |
| WhatsApp bugs | 190 | 9:15 |
| Close | 140 | 10:00 |
| **Total** | **~1,550** | |

If it runs long, cut from **Four decisions** or **How it works**.
Never cut the Hook, the Re-hook at 5:00, or the Close.

## Shot list

| # | Shot | Where | Notes |
|---|---|---|---|
| 1 | Title cards (₹455, "Ab galtiyan") | any editor | plain text, high contrast |
| 2 | Terminal / GitHub profile | laptop | for the intro |
| 3 | myScheme.gov.in scrolling | laptop | show the English + the filters |
| 4 | Saans repo, then closing the tab | laptop | the "I threw it away" beat |
| 5 | `sathi/rules/engine.py`, `tests/test_rules.py` | laptop | four decisions |
| 6 | WhatsApp conversation, full flow | phone | **needs scheme sign-off first** |
| 7 | The state question rendering as a list | phone | WhatsApp-specific, nice detail |
| 8 | `docs/BUILD_LOG.md` bugs section | laptop | scroll slowly |
| 9 | `is_verified` property | laptop | the worst bug |
| 10 | ssh + `systemctl status sathi` | laptop | deployment proof |
| 11 | Meta dashboard, `subscribed_apps` response | laptop | the lying toggle |
| 12 | sqlite thread error in the journal | laptop | second bug |

**Blocking:** shot 6 needs the schemes signed off, otherwise the app answers
"pata nahi" to everything and the results section cannot be filmed. Keep **one**
scheme unsigned on purpose so you also capture a real "pata nahi" for the close.

## Title options

1. Maine ek app banaya jo "mujhe nahi pata" bolta hai — aur wahi iska point hai
2. 6 bugs in the first hour — building a real product with no job, no team
3. Kotdwara se AWS tak: ek project ki poori kahani

## Description skeleton

```
Maine ek WhatsApp aur Telegram bot banaya jo India ke unorganised workers ko
batata hai ki unhe kaunsi sarkari schemes mil sakti hain — Hindi mein, buttons
par, bina kuch type kiye.

Is video mein poori kahani hai: kyun banaya, kaise banaya, kya toota, aur kya
aaj bhi galat hai.

Code (open source): github.com/avinashnegi1999/yojana-sathi
Build log (saari galtiyan): github.com/avinashnegi1999/scheme-sathi-handbook
Telegram: @YojanaSathiBot

00:00 ₹455
00:15 Main kaun hoon
01:00 Problem kya hai
02:00 Jo project maine phenk diya
02:45 Chaar decisions
04:00 Kaise kaam karta hai
05:20 Kya toota
06:50 AWS par deploy
08:00 WhatsApp ke do bugs
09:15 Kya abhi bhi galat hai
```

**Tags:** python, backend, aws ec2, whatsapp cloud api, telegram bot, build in
public, indian developer, self taught developer, side project

---

# NUMBERS THIS SCRIPT USES — all verified 2026-09-07

₹455/day male and ₹315/day female casual labourer · 3 schemes · PMSBY ₹2,00,000
cover for ₹20/year · PM-SYM ₹3,000/month from age 60 · e-Shram ₹0, a gateway ·
zero third-party dependencies · 2 channels · AWS ap-south-1 · ~$10/month ·
Saans ≈ 3,500 lines · 6 bugs in the first hour.

## Do NOT add to this script

- **Any count of unorganised workers in India.** Not sourced anywhere in the
  repo. AI summaries of this project have invented "44M+" before.
- **"Zero persistence"** — false, there is a database with two tables.
- **"No hallucinations" / "0% hallucination"** — the model is optional and off.
  That is a design choice, not a proof.
- **"Works without a smartphone"** — both channels need one.
- **PM-KISAN, Ayushman Bharat** — not in this project.

Quoting an unverifiable number is the exact failure this project exists to
prevent. A public video is the worst possible place to slip.

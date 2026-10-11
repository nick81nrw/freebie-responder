# Freebie-Responder für Instagram – Recherche (Stand: Oktober 2026)

## 1. Offizieller technischer Weg

Einziger erlaubter Weg ist die **Instagram Platform API (Meta Graph API)**. Inoffizielle
Bots/Scraper (Login-Automatisierung, z. B. instagrapi) verstoßen gegen die Nutzungsbedingungen
und führen zu Action-Blocks oder Account-Sperren → **nicht verwenden**.

### Voraussetzungen
| Punkt | Anforderung |
|---|---|
| Instagram-Konto | Professional (Business oder Creator), **öffentlich** |
| Login-Variante | *Instagram API with Instagram Login* (`graph.instagram.com`, keine FB-Seite nötig) **oder** *with Facebook Login* (`graph.facebook.com`, verknüpfte FB-Seite) |
| Meta-App | Developer-Account, App im Modus **Live** |
| Business-Verifizierung | Erforderlich (Handelsregister/Gewerbe-Nachweis) |
| App Review / Advanced Access | Für `comments`-Webhooks und Messaging-Permissions nötig (Screencast + Begründung) |
| Permissions (IG Login) | `instagram_business_basic`, `instagram_business_manage_comments`, `instagram_business_manage_messages` |
| Infrastruktur | Öffentlich erreichbarer HTTPS-Webhook-Endpoint, Token-Speicher, Queue für Rate-Limits |

Tipp: Solange die App nur für das **eigene** Konto genutzt wird, kann man im Development-Modus
mit App-Rollen (Admin/Tester) testen; für den Produktivbetrieb mit Webhooks muss die App Live sein.

### Ablauf „Kommentar → DM" (Comment-to-DM)
1. Webhook `comments` liefert Kommentar (Text, Media-ID, Kommentar-ID, IG-scoped User-ID).
2. Keyword-Prüfung im eigenen Backend.
3. **Private Reply** an `comment_id` senden (`POST /<IG_ID>/messages` mit `recipient: {comment_id}`).
4. Optional öffentliche Antwort auf den Kommentar („Schau in deine DMs 📩").
5. Erst wenn der User **antwortet oder einen Button klickt** (`messaging_postbacks`), öffnet sich
   das 24h-Fenster → dann Follow-Check und Versand des Download-Links.

## 2. Prüfbarkeit der Bedingungen

| Bedingung | Per API prüfbar? | Wie |
|---|---|---|
| **Kommentar mit Keyword** | ✅ Ja | Webhook `comments`, Text-Matching im Backend |
| **Gefolgt** | ✅ Ja (mit Einschränkung) | User Profile API: Feld `is_user_follow_business`. Funktioniert nur nach Consent, d. h. nachdem der User eine Nachricht geschickt / Button geklickt hat. Typisches Muster: „Folgst du mir? → [Ja, erledigt]" → Check → Link oder erneute Bitte |
| **Story-Share mit @Mention** | ✅ Teilweise | Kommt als DM-Event (`messages`, Attachment-Typ `story_mention`) – nur wenn der User dich in der Story markiert |
| **Geliked** | ❌ Nein | Graph API liefert nur `like_count`, keine Liste der Liker, kein Like-Webhook |
| **Geteilt (DM/Story ohne Mention)** | ❌ Nein | Keine API, nur aggregierte Insights (`shares`) |
| **Gespeichert** | ❌ Nein | Nur aggregiert in Insights |
| **DM mit Keyword** | ✅ Ja | Webhook `messages` |
| **Freunde markiert** | ✅ Indirekt | Kommentartext auf `@username` parsen |

**Fazit:** Realistisch automatisierbar sind *Keyword-Kommentar*, *Follow* und *Story-Mention*.
Like- und Share-Pflichten lassen sich nicht verifizieren – entweder weglassen oder nur „auf
Vertrauensbasis" erwähnen. Kein seriöses Tool kann das (alle Anbieter nutzen dieselbe API).

## 3. Limits

| Limit | Wert | Quelle |
|---|---|---|
| Private Replies auf Post-/Reel-Kommentare | **750 / Stunde / Account** | Meta Rate-Limiting-Doku |
| Private Replies auf Live-Kommentare | 100 / Sekunde, nur während Live | Meta |
| Conversations API | 2 Calls / Sekunde / Account | Meta |
| Private Reply pro Kommentar | **genau 1** | Meta |
| Zeitfenster Private Reply | **7 Tage** nach Kommentar | Meta |
| Folge-Nachrichten | nur innerhalb **24h** nach User-Antwort | Meta |
| Human-Agent-Tag | bis 7 Tage, nur für echten Support, nicht für Marketing | Meta |
| „200 DMs/Stunde" | oft zitiert, **nicht** in Meta-Doku belegt – eher Tool-Konvention | Drittanbieter |

Weitere Hinweise:
- Private Reply landet im **Postfach**, wenn der User folgt, sonst in **Nachrichtenanfragen**.
- Eine Private Reply öffnet **nicht** das 24h-Fenster – erst eine Antwort/ein Button-Klick des Users.
- Bei viralen Posts (>750 Kommentare/h) braucht man eine Queue; die 7-Tage-Frist gibt Puffer.
- Meta drosselt bei Spam-Signalen (identische Texte, viele Meldungen) – Nachrichten leicht variieren.

## 4. Alternative Produkte (Stand 2026, Preise lt. Anbieter/Vergleichsseiten – vor Kauf prüfen)

| Tool | Preismodell | Free | Einstieg | Follow-Gate | Besonderheiten |
|---|---|---|---|---|---|
| **ManyChat** | pro aktivem Kontakt | 25 Kontakte (Free stark begrenzt) | ca. 14–17 $/Monat (250 Kontakte), steigt mit Kontakten | ✅ | Marktführer, offizieller Meta-Partner, Flow-Builder, Integrationen (Zapier, Make, Mailchimp, Google Sheets), auch FB/WhatsApp/TikTok; teuer bei viralen Posts |
| **LinkDM** | nach DMs | 1.000 DMs/Monat | 19 $/Monat (25.000 DMs, 3 Accounts) | ✅ | Speziell Comment-to-DM, simpel |
| **CreatorFlow** | nach DMs | 500 DMs/Monat | 15 $ (5.000) / 30 $ (10.000) | ✅ | Pauschal, keine Kontaktgebühren |
| **Zorcha** | – | großzügig/„unlimited" | Bezahlpläne unklar | ✅ | Neuer Anbieter, Preisstatus unsicher |
| **Chatfuel / InstantDM / SuperProfile u. a.** | variabel | teils | ~10–30 $ | meist ✅ | Viele Clones mit gleichem Funktionsumfang |

Alle nutzen dieselbe offizielle API → **gleiche technischen Grenzen** (kein Like-/Share-Check).
Unterschiede liegen in Preis, Flow-Builder, E-Mail-Erfassung, CRM-/Newsletter-Anbindung, Analytics.

## 5. Eigenbau vs. Tool

| | Eigenbau | SaaS-Tool |
|---|---|---|
| Aufwand | Meta-App, Business-Verifizierung, App Review (Tage–Wochen), Hosting | Minuten |
| Kosten | Hosting (~0–10 €/Monat) | ~15–50 €/Monat, bei ManyChat skalierend |
| Datenschutz | volle Kontrolle, Hosting in der EU möglich | US-Anbieter → AVV/DSGVO prüfen |
| Flexibilität | beliebige Logik (z. B. Download-Link mit Token/Ablauf, E-Mail-Opt-in, eigenes CRM) | im Rahmen des Flow-Builders |
| Risiko | App Review kann abgelehnt werden | Abhängigkeit vom Anbieter |

**Empfehlung:** Für eine einzelne Aktion/ein einzelnes Profil ist ein Tool (LinkDM/CreatorFlow
günstig, ManyChat bei Bedarf an Integrationen) am schnellsten. Ein Eigenbau lohnt sich bei
mehreren Aktionen, eigener Lead-Datenbank, DSGVO-Anforderungen oder individuellen Download-Links
(z. B. personalisierte, ablaufende Links).

## 6. Rechtliches (DE/EU) – Kurzcheck
- **DSGVO:** Verarbeitung von IG-ID/Username/E-Mail → Datenschutzerklärung anpassen, ggf. AVV mit Tool-Anbieter.
- **E-Mail-Erfassung für Newsletter:** Double-Opt-in, Kopplungsverbot beachten (Freebie nicht zwingend an Newsletter koppeln ohne klare Info).
- **Meta Promotion Guidelines:** Keine falschen Markierungen erzwingen; bei Gewinnspielen Teilnahmebedingungen + Haftungsfreistellung von Meta.
- **Werbekennzeichnung/Impressum** auf der Download-Seite.

## Quellen
- Meta: Private Replies – https://developers.facebook.com/docs/instagram-platform/private-replies
- Meta: Rate Limiting – https://developers.facebook.com/docs/graph-api/overview/rate-limiting
- Meta: Webhooks – https://developers.facebook.com/docs/instagram-platform/webhooks
- Meta: User Profile API (`is_user_follow_business`) – https://developers.facebook.com/docs/instagram-platform/instagram-api-with-instagram-login/messaging-api/user-profile
- Limits-Übersicht (Drittanbieter): https://www.reachlee.co/guides/instagram-dm-limits/ , https://creatorlanehq.com/blog/instagram-dm-auto-reply-rate-limits-2026
- Tool-Vergleiche: https://www.blotato.com/blog/manychat-alternatives , https://instantdm.com/blog/top-15-manychat-alternatives-for-instagram-automation-in-2026 , https://www.tryunlockdm.com/blog/best-instagram-dm-automation-tools-2026

# Sources, data and licenses

School Air Quality Monitoring exists thanks to the hardware and reference data of other projects — some donated equipment, others publish the data we use as context. This page spells out what's used from each one, under what license, and how we plan to use (and give back) that data. It isn't a formal legal document — it's a transparency note, written so that anyone (including the providers themselves) can see at a glance what we do with what they gave us.

---

## PurpleAir

[purpleair.com](https://www.purpleair.com) makes the **PurpleAir Flex** sensors that form the core of this network. They were born in 2015, when their founder, Adrian Dybwad, wanted to measure the dust a quarry near his home in Draper, Utah was kicking up, and found nothing he could buy that didn't cost thousands of dollars:

> "I was curious and wanted to know how much dust was blowing in. But there wasn't anything I could buy to see."

That's how a low-cost sensor came about, meant for anyone to answer that same question for themselves. The philosophy holds today:

> "PurpleAir is about the community. We're people worldwide with a common desire to know what we're breathing."

### How they're used here

The network's **5 PurpleAir Flex sensors** were a donation from the [PurpleAir Collective](https://community.purpleair.com/t/purpleair-collective-june-july-2024/8771) (mid-2024 call), based on a statement of need the author submitted — the project placed 2nd in that round. Data is polled every 15 minutes via PurpleAir's official API (`api.purpleair.com`) and stored in a dedicated database (Cloudflare D1) so history and reports can be offered — PurpleAir only retains short windows on its own platform.

Since PurpleAir started charging for API access, that access is no longer free by default — today it's paid for out of the author's own credits. A no-cost arrangement for this specific project is being worked out with PurpleAir, given it's an educational use and the sensors were received as a donation.

### License

PurpleAir's map data and real-time API data are published under its own [data license](https://www.purpleair.com/license) (non-commercial use, with attribution). This project is entirely educational and non-profit, and credits PurpleAir as a source on every sensor card and in the site's footer. For the full, current text, the reference is always the license page linked above.

**Links:** [About](https://www.purpleair.com/about) · [Data license](https://www.purpleair.com/license) · [Terms of service](https://www.purpleair.com/policies/terms-of-service) · [Privacy](https://www.purpleair.com/policies/privacy-policy)

---

## AirGradient

[airgradient.com](https://www.airgradient.com) makes the **Open Air** sensors that add CO2, VOC and NOx readings to the network on top of particulate matter. It's open hardware and firmware: they publish designs and code under a **CC BY-SA 4.0** license, and explicitly invite people to modify them.

They have a strong stance on who owns air quality data:

> "Always read the fine print on data ownership. Choose monitors where YOU own the data and can share it freely." — AirGradient's website

Their CEO, Achim Haug, even frames it in ethical terms: restricting data ownership is *"wrong and against humanity's best interests,"* even though a company can still stay profitable while keeping data open. They even have an [interactive quiz](https://www.airgradient.com/aq-data-ownership-quiz/) to help spot restrictive fine print in other manufacturers' terms.

### How they're used here

The network's **2 AirGradient Open Air sensors** are active, currently connected at a private home as a trial while it's decided which institutions they'll be installed at. One arrived as a donation through the OpenAQ Community Ambassadors program and the other was bought directly from AirGradient; both are managed the same way, inside AirGradient's own ecosystem. Data is polled through AirGradient's own API and stored in the same D1 database as PurpleAir, with a `proveedor` field distinguishing the source.

Since both are registered in AirGradient's ecosystem, they're already visible on AirGradient's [public map](https://map.airgradient.com) too — with no extra step needed on our end.

### License

AirGradient's hardware and firmware are **CC BY-SA 4.0** (free use and modification with attribution). The data its monitors generate stays under the control of whoever operates them — in this case, the project — which decides whether to publish it. Here it's published through School Air Quality Monitoring's own API, in the same open-data spirit AirGradient promotes.

**Links:** [Privacy](https://www.airgradient.com/privacy-policy/) · [Terms](https://www.airgradient.com/terms-conditions/) · [Data ownership quiz](https://www.airgradient.com/aq-data-ownership-quiz/)

---

## OpenAQ

[openaq.org](https://openaq.org) is a non-profit whose mission is to open up access to air quality data worldwide — it "fights air inequality by opening up air quality data." It doesn't sell data or user information, taking a stance similar to AirGradient's: environmental data is a public good.

### How it's used here

Today OpenAQ is mostly a design and project-philosophy reference — there's no active data integration between School Air Quality Monitoring and OpenAQ yet. AirGradient offers the option to publish its monitors' data to OpenAQ (it's the operator's choice); it's something worth evaluating in the future for the network's 2 AirGradient sensors. The PurpleAir sensors, on the other hand, don't reach OpenAQ today: since PurpleAir started monetizing its API access, OpenAQ stopped picking up its monitors.

**Links:** [Privacy](https://openaq.org/privacy/) · [Terms](https://openaq.org/terms/)

---

## The data this project generates

Following the same philosophy as PurpleAir, AirGradient and OpenAQ, the data School Air Quality Monitoring generates and publishes (sensor metadata, historical readings) is available with no authentication or sign-up through the project's [public API](https://aq.lemeit.ar/api.html) — read-only, open CORS, meant for anyone to consume directly. If you reuse this data publicly, we'd appreciate a credit to [app.lemeit.ar/aq](https://app.lemeit.ar/aq).

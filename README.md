# Saiko catalogue

Reference data for [Saiko](https://github.com/saiko-app), an offline-first journal and reference app for iPhone.

The catalogue keeps sources, review states and gaps alongside the information. Personal journal records are never part of it.

## Releases

**The first release is being prepared. No data files are available yet.**

Approved releases will appear under [Releases](https://github.com/saiko-app/catalogue/releases), with versioned data files, source credit, licences and change notes. Signed app updates use `manifest.json`, `manifest.sig` and `catalogue.sqlite`. See the [update format](spec/update-v1.md) and [database schema](spec/catalogue-schema-v1.md).

The first release waits for the update code and signing setup. The app's update format currently covers one primary database; separately licensed companion files keep their own packaging and are not silently merged into it.

## Sources

Releases are planned to include PsychonautWiki reference data, **TripSit drug and combination data**, and other available sources under their applicable terms. We aim to share as much data as those terms allow. The [source registry](sources.json) distinguishes planned sources from excluded material; each release will list exactly what it contains.

TripSit data is credited to [TripSit](https://combo.tripsit.me/), with a backlink wherever it is shown. It is provided under [TripSit's non-commercial terms](https://wiki.tripsit.me/wiki/Drug_combinations#Use_.26_Attribution). This information is only a quick overview and reference. Do your own research before making a decision.

PsychonautWiki's structured API data uses CC BY 4.0; its wiki draft data uses CC BY-SA 4.0. Wikidata's structured data uses CC0, and ChEBI data uses CC BY 4.0. Other sources keep their own rights. Read the [attribution](ATTRIBUTION.md) and [licence notices](LICENSES/README.md) before reuse.

## What the data means

Every claim retains its source and review state. Versions, changes and applicability are recorded, and unknown information stays unknown. Sources that copy one another do not count as independent confirmation.

Some values are unreviewed, disputed or withdrawn. A signed release identifies the bytes Saiko published; it does not prove that a claim is correct or applies to an individual. Missing information does not establish safety. The catalogue is a reference, not personalised medical advice or an assurance of safe use.

## Reuse

Keep each file's source credit, links, licence and change notices. Data retains its own licence; there is no single licence that overrides every source. TripSit data remains non-commercial and separately removable. Other files may permit commercial reuse under their own terms.

The repository contains distribution documentation. The full authoring pipeline and research evidence are maintained separately. Release notes will identify any available reproduction materials.

## Corrections and privacy

To report a catalogue issue, identify the release, substance, route and source or claim ID, and link supporting public evidence. Do not post journal exports, personal health information or private research.

Downloading reference data from GitHub exposes normal request metadata to GitHub and its delivery providers. Catalogue downloads do not include personal journal records, and this repository makes no zero-logging promise.

Higher score is better
Score range: 0 to 1

Problem Description
Writing Zürich Street Names from Their Naming Notes
Overview
Each row is a street, square, path, stair or bridge of the city of Zürich, given only by the note the city keeps on why its name was chosen, written in German, and for some rows the text of its street-name plaque. Your task is to write the street's official name exactly as it appears on the sign, together with the city district (Kreis) it lies in and the year of the earliest recorded decision or attestation of the name.

Zürich has named its streets for two centuries, and its street-name book records the reason for each name. A note may quote an old field name in dialect and its meaning, give the life dates and role of the person honoured, recall a house, inn, mill, factory, brook or event, or simply place the name in a theme. The names themselves follow local habits: some field names keep their dialect form and others are modernised, a name may end in strasse, weg, gasse, platz, steig, rain, halde or in no street word at all (Im Hegi, Kreuzwiesen), and people are honoured by surname alone or by full name (Burriweg, Else-Züblin-Strasse). Large parts of today's city were separate municipalities until they were merged into Zürich in 1893 and 1934, and their councils named many of the older streets.

Every answer comes from the city's register: the name and Kreis of the street, and the smallest year among the decisions and attestations recorded for the name. Some names were first recorded long before the person or building in the current note: the note can describe a later dedication of an older name.

A note does not always determine its answer uniquely. The same field name can become a ...strasse in one place and a ...weg in another, a note may not say which part of the city the street is in, and the year is not always written in it. This is a prediction task: a solution predicts the most likely name, Kreis and year from the note, the training data and general knowledge, and the scoring gives partial credit for the right street word, an overlapping Kreis and a year within 20 years. A perfect score is not expected.

Wherever the street's own name appeared in its note or plaque text, it was replaced by [Name]. Where another street's name shares its stem with a test street (a ...strasse and a ...weg named after the same field, for example), that name was replaced by [street] in every row. Related streets are all on the same side of the train/test split: streets that share a name stem, a note, or a passage of eight or more words in their notes.

Dataset
Files:

train.csv — 1,553 streets with their name, Kreis and year

test.csv — 524 streets (440 groups of related streets) without them

sample_submission.csv — the submission format, filled with placeholder values

Columns of train.csv and test.csv:

id (string): row identifier, such as zs_016e2dc34087

text (string): the naming note in German. When the street has a plaque, its text follows after a blank line and the label "Plaque:", with the plaque's own line breaks. Notes run from 40 to about 1,900 characters (median about 200).

Additional columns of train.csv:

name (string): the official street name, for example Erligatterweg, Kreuzwiesen, Im Hegi, Gänziloobrücke or Susanna-Gossweiler-Platz

kreis (string): the Kreis number, 1 to 12. A street running through several districts lists them joined by "+", for example 6+11 or 3+4+9.

year (integer): the year of the earliest recorded naming decision or attestation of the name, from 1794 to 2025. Decisions are those of the city council and, before the mergers of 1893 and 1934, of the councils of the municipalities that now form the city; attestations come from early house and street directories, address books and plans.

A training row, for illustration:


text: Flurname «Erli» (ältere Form von «Erle», altdeutsch «Erila») zusammengesetzt mit «Gatter» bedeutet vermutlich eingezäuntes, Landstück mit Durchgangstor bei einer Erle oder einem Erlenwäldchen (1632 «bim Erli gatter», 1642 «im Erligarten»).

name: Erligatterweg

kreis: 2

year: 1950

Submission
Submit a CSV with one row per test id and four columns:

id (string): the test id

name (string): the official street name

kreis (string): the Kreis number, or several joined by "+"

year (integer): the year of the earliest recorded naming decision or attestation


id,name,kreis,year

zs_0a1b2c3d4e5f,Erligatterweg,2,1950

zs_6a7b8c9d0e1f,Hamamelisweg,6+11,2012

Evaluation
Each test row gets a score between 0 and 1:


row score = 0.50 × N + 0.15 × E + 0.15 × K + 0.20 × Y

N = 1 if the predicted name equals the true name after normalisation, else 0

E = 1 if the predicted name is not empty and ends in the same street word as the true name, else 0

K = |P ∩ T| / |P ∪ T|, where P and T are the predicted and true sets of Kreis numbers

Y = max(0, 1 - |predicted year - true year| / 20)

score = mean row score over all test rows

Normalisation for N: Unicode NFC, case-folded, ß written as ss, and spaces, hyphens, full stops and apostrophes removed. So "Erligatter-Weg" matches Erligatterweg, but "Erligatterstrasse" does not.

Street words for E: strasse, weg, gasse, gässli, platz, steig, steg, brücke, quai, ring, rain, halde, hof, park, anlage, promenade, allee, treppe, terrasse, graben, rank, garten, pfad, stieg, trail. The ending of a name is the longest of these words that the normalised name ends with, or "none" if it ends with none of them (as Kreuzwiesen and Im Hegi do). E is 1 when the two endings are equal.

K reads every number from 1 to 12 in the kreis value, so "6+11", "6/11" and "Kreis 6, 11" are read the same way. Y reads the first number of three or four digits in the year value.

Worked examples:

True Erligatterweg, Kreis 2, 1950; predicted Erligatter-Weg, 2, 1943. N = 1, E = 1 (weg), K = 1, Y = 1 - 7/20 = 0.65. Row score = 0.50 + 0.15 + 0.15 + 0.20 × 0.65 = 0.93.

True Hamamelisweg, Kreis 6+11, 2012; predicted Hamamelisstrasse, 11, 1990. N = 0, E = 0 (strasse against weg), K = 1/2, Y = max(0, 1 - 22/20) = 0. Row score = 0.15 × 0.5 = 0.075.

True Kreuzwiesen, Kreis 12, 1949; predicted Kreuzwiesenweg, 12, 1949. N = 0, E = 0 (weg against none), K = 1, Y = 1. Row score = 0.15 + 0.20 = 0.35.

A field that is empty or cannot be read scores 0 for its part of the row, and grading continues. The submission must contain every test id exactly once. The score lies between 0 and 1; higher is better.

What Not To Use
A solution may use the provided files, general knowledge of German, Swiss German, geography and history, and general-purpose pretrained models. It may not consult any source that lists Zürich's streets.

Do not look the streets up: no web search, and no maps, street directories, gazetteers, address databases, the city's street-name pages or books on Zürich street names, nor copies of any of them.

Do not hand-label test rows or hard-code test ids or answers.

Hosted model APIs and other remote services (for example chat or completion endpoints) may not be called to produce predictions.

Open-weight pretrained models (language models, embeddings) may be used and fine-tuned, as long as they were not trained or tuned specifically on Zürich street names or street-name records. What such a model learned in general pretraining may be used.

 

Expected Output
Your script receives the public dataset directory and exact submission CSV path as two positional arguments.
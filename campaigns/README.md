# campaigns

Realistic, ordered attack chains — the opposite of running techniques at random. Each
campaign simulates one coherent intrusion from initial execution through to impact, and
every stage links out to the real detection work in `detections/`.

This folder is the narrative view of the lab. `detections/` is where the actual evidence
and rules live; a campaign never duplicates that content, it just tells the story of the
order techniques were run in and why.
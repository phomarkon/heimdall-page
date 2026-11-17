# Heimdall project page

Project page for "Heimdall: Only the Safe Shall Pass. A Conformal Verifier Between Autonomous Bidders and the Nordic Electricity Balancing Market", the BSc Software Engineering thesis of Phongsakon Mark Konrad and Tim Lukas Adam (equal contribution) at the University of Southern Denmark, Sønderborg, with Danfoss A/S as industrial partner (2026). The page is served at https://phomarkon.github.io/heimdall-page/.

This repository holds the built static page only. The code, data pipeline and run configurations are at https://github.com/phomarkon/heimdall-src, and the forecaster checkpoints are at https://huggingface.co/Phongsakon/heimdall.

The `app/` folder is a static build of the thesis operator dashboard from the code release, replaying the curated runs that ship with it. Its run data is the run-view API output for those runs, stored gzipped under `app/data/`. Saving specs and templates needs the local stack and is disabled here.

Every plotted value on the main page comes from the thesis or from the stored results in the code release. The gate figure on Sheet 02 is a labelled illustration, and the simulator runs the thesis calibrators and gate on an invented price stream. It is labelled as simulated and shows no thesis results.

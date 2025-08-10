# dctm-cmis-upload-form
A minimal Node.js + HTML web application that captures a user’s details and file attachment, then uploads them to an OpenText DCTM repository using the CMIS Browser Binding API. Includes a responsive frontend form, secure backend bridge.

# Mostafa Mansour, Solutions Consultant - Opentext
# Step-by-step deployment guide


Step 1 — Install verify CMIS 
Deploy CMIS webapp

Drop emc-cmis.war into Tomcat’s webapps/ and start Tomcat.

If required, place a dfc.properties in the CMIS webapp’s WEB-INF/classes/ to point to your docbroker and repo. (Typical entries: dfc.docbroker.host[0], dfc.docbroker.port[0], and the global registry values.)

Many deployments expose CMIS at:
http(s)://<host>:<port>/emc-cmis/browser (Browser Binding, CMIS 1.1). 
dbi services

Quick sanity test

Open the browser URL above; you should get repository JSON.

The repositoryId is usually your docbase name. (The Browser Binding URL shape and semantics are defined by the CMIS 1.1 spec.) 
OASIS Open

Why CMIS Browser Binding? It’s JSON over HTTP(S) using simple POST/GET with form parameters like cmisaction=createDocument, propertyId[n], propertyValue[n], etc. 
OASIS Open
GitHub

Step 2 — (Optional) Install & verify Documentum REST (native API)
If you prefer native API later (or for health checks):

Deploy dctm-rest.war to Tomcat and start it.

Verify: open
http(s)://<host>:<port>/dctm-rest/repositories
You should see repo listings; basic auth works out of the box. 
Stack Overflow
FME US, llc

(You can keep REST for admin/diagnostics and still upload via CMIS.)

Step 3 — Prepare a target folder & account in Documentum
Create (or pick) a cabinet/folder where uploads will land (e.g., /Cabinets/Incoming/Onboarding).

Ensure your service account can create documents there (ACLs / permissions).

Note the folder’s r_object_id (you’ll use it as CMIS_FOLDER_ID).
(You can fetch IDs via DA, DQL, or REST if you prefer.)

Step 4 — Configure the Node.js bridge (the code I gave you)
Put the earlier files in place (public/index.html, server.js, package.json).

Create .env and fill it like so:

ini
Copy
Edit
CMIS_URL=https://<host>:<port>/emc-cmis/browser
CMIS_REPOSITORY_ID=<your_repo_name>
CMIS_FOLDER_ID=<target_folder_r_object_id>
CMIS_USERNAME=<service_user>
CMIS_PASSWORD=<password>

# optional if you have a custom type/attributes
CMIS_OBJECT_TYPE_ID=cmis:document
CUSTOM_NAME_PROP=
CUSTOM_ID_PROP=
PORT=3000
Install & run:

bash
Copy
Edit
npm install
npm start
# open http://localhost:****
Submit the form—your Name, ID Number, and file are posted to /api/submit, and the backend performs a CMIS Browser “createDocument” into the folder you configured. (The parameters and flow mirror the CMIS Browser Binding spec.) 
OASIS Open
GitHub

Step 5 — Smoke test CMIS directly (handy for troubleshooting)
Try a raw curl against your CMIS folder to prove the path & creds:

bash
Copy
Edit
curl -u USER:PASS \
  -F 'cmisaction=createDocument' \
  -F 'propertyId[0]=cmis:objectTypeId' -F 'propertyValue[0]=cmis:document' \
  -F 'propertyId[1]=cmis:name' -F 'propertyValue[1]=test_upload.txt' \
  -F 'content=@/path/to/local/file.txt;type=text/plain' \
  -F 'versioningState=none' \
  -F 'succinct=true' \
  "https://<host>:<port>/emc-cmis/browser/<REPO_ID>/id/<FOLDER_ID>"
If this succeeds, the Node app will, too. (Those fields come straight from Browser Binding semantics.) 
OASIS Open
GitHub

Step 6 — (Optional) Use native Documentum REST instead of CMIS
You can swap the Node handler to call Documentum REST. The REST webapp exposes the repo, folders, and upload links under /dctm-rest (discoverable via HATEOAS). A common pattern is to POST multipart to the folder’s “documents” link with the content stream and JSON metadata; authentication is basic unless you’ve enabled SSO. See the official REST API references/samples for exact link relations and payloads. 
OpenText Developer
GitHub

Step 7 — Production hardening (recommended)
TLS end-to-end (terminate at a reverse proxy if needed).

Least-privilege service account; deny delete if not required.

File size and MIME limits on the Node API (and Tomcat).

Virus scanning / DLP if your policy requires it.

Observability: add request logging and map CMIS errors to friendly messages.

Backups and retention in Documentum per your compliance rules.

Common pitfalls & fixes
401/403 from CMIS: wrong creds or missing folder permissions.

404 on /emc-cmis/browser: CMIS webapp not deployed or context path differs. 
dbi services

Repo not listed: CMIS/REST can’t reach Content Server (check dfc.properties/docbroker)


/**
* End!
 */
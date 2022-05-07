<script>
  // Load Google API config file.
  import config from "./client_secret.json";

  /* exported gapiLoaded */
  /* exported gisLoaded */
  /* exported handleAuthClick */
  /* exported handleSignoutClick */

  // TODO(developer): Set to client ID and API key from the Developer Console
  const CLIENT_ID = config.web.client_id;
  const API_KEY = "<YOUR_API_KEY>";

  // Discovery doc URL for APIs used by the quickstart
  const DISCOVERY_DOC =
    "https://www.googleapis.com/discovery/v1/apis/calendar/v3/rest";

  // Authorization scopes required by the API; multiple scopes can be
  // included, separated by spaces.
  const SCOPES = "https://www.googleapis.com/auth/calendar.readonly";

  let tokenClient;
  let gapiInited = false;
  let gisInited = false;

  let authorize_button_text = "Authorize";
  let signout_button_text = "Sign Out";
  let content_text = "";

  // document.getElementById("authorize_button").style.visibility = "hidden";
  let authorize_button_visibility = "hidden";
  // document.getElementById("signout_button").style.visibility = "hidden";
  let signout_button_visibility = "hidden";
  /**
   * Callback after api.js is loaded.
   */
  function gapiLoaded() {
    gapi.load("client", intializeGapiClient);
  }

  /**
   * Callback after the API client is loaded. Loads the
   * discovery doc to initialize the API.
   */
  async function intializeGapiClient() {
    await gapi.client.init({
      apiKey: API_KEY,
      discoveryDocs: [DISCOVERY_DOC],
    });
    gapiInited = true;
    maybeEnableButtons();
  }

  /**
   * Callback after Google Identity Services are loaded.
   */
  function gisLoaded() {
    tokenClient = google.accounts.oauth2.initTokenClient({
      client_id: CLIENT_ID,
      scope: SCOPES,
      callback: "", // defined later
    });
    gisInited = true;
    maybeEnableButtons();
  }

  /**
   * Enables user interaction after all libraries are loaded.
   */
  function maybeEnableButtons() {
    if (gapiInited && gisInited) {
      // document.getElementById("authorize_button").style.visibility = "visible";
      authorize_button_visibility = "visible";
    }
  }

  /**
   *  Sign in the user upon button click.
   */
  function handleAuthClick() {
    tokenClient.callback = async (resp) => {
      if (resp.error !== undefined) {
        throw resp;
      }
      // document.getElementById("signout_button").style.visibility = "visible";
      signout_button_visibility = "visible";
      // document.getElementById("authorize_button").innerText = "Refresh";
      authorize_button_text = "Refresh";
      await listUpcomingEvents();
    };

    if (gapi.client.getToken() === null) {
      // Prompt the user to select an Google Account and asked for consent to share their data
      // when establishing a new session.
      tokenClient.requestAccessToken({ prompt: "consent" });
    } else {
      // Skip display of account chooser and consent dialog for an existing session.
      tokenClient.requestAccessToken({ prompt: "" });
    }
  }

  /**
   *  Sign out the user upon button click.
   */
  function handleSignoutClick() {
    const token = gapi.client.getToken();
    if (token !== null) {
      // CORS issue only when revoking Token by calling (Google Identity Services) GIS's google.accounts.oauth2.revoke
      // https://stackoverflow.com/questions/71532935/cors-issue-only-when-revoking-token-by-calling-google-identity-services-giss
      google.accounts.oauth2.revoke(token.access_token);
      gapi.client.setToken("");
      // document.getElementById("content").innerText = "";
      content_text = "";
      // document.getElementById("authorize_button").innerText = "Authorize";
      authorize_button_text = "Authorize";
      // document.getElementById("signout_button").style.visibility = "hidden";
      signout_button_visibility = "hidden";
    }
  }

  /**
   * Print the summary and start datetime/date of the next ten events in
   * the authorized user's calendar. If no events are found an
   * appropriate message is printed.
   */
  async function listUpcomingEvents() {
    let response;
    try {
      const request = {
        calendarId: "primary",
        timeMin: new Date().toISOString(),
        showDeleted: false,
        singleEvents: true,
        maxResults: 10,
        orderBy: "startTime",
      };
      response = await gapi.client.calendar.events.list(request);
    } catch (err) {
      // document.getElementById("content").innerText = err.message;
      content_text = err.message;
      return;
    }

    const events = response.result.items;
    if (!events || events.length == 0) {
      // document.getElementById("content").innerText = "No events found.";
      content_text = "No events found.";
      return;
    }
    // Flatten to string to display
    const output = events.reduce(
      (str, event) =>
        `${str}${event.summary} (${
          event.start.dateTime || event.start.date
        })\n`,
      "Events:\n"
    );
    // document.getElementById("content").innerText = output;
    content_text = output;
  }
</script>

<svelte:head>
  <script src="https://apis.google.com/js/api.js" on:load={gapiLoaded}></script>
  <script
    src="https://accounts.google.com/gsi/client"
    on:load={gisLoaded}></script>
</svelte:head>

<main class="container">
  <p>Google Calendar API Quickstart</p>

  <!--Add buttons to initiate auth sequence and sign out-->
  <button
    id="authorize_button"
    on:click={handleAuthClick}
    style="visibility: {authorize_button_visibility};"
    >{authorize_button_text}</button
  >
  <button
    id="signout_button"
    on:click={handleSignoutClick}
    style="visibility: {signout_button_visibility};"
    >{signout_button_text}</button
  >

  <pre id="content" style="white-space: pre-wrap;">{content_text}</pre>
</main>

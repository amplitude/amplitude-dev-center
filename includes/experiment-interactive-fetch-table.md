<!-- markdown-link-check-disable -->
!!!warning "Potential data changes"
    This example makes real requests to the API and can potentially change the data in your Amplitude project. We recommend using a development project when testing APIs.

<pre>
<code>curl --request GET \
     --url '<span id='curl_url'></span><span id='curl_user_id'></span>' \
     --header 'Authorization: Api-Key <span id='curl_deployment_key'></span>'
</code>
</pre>

| <div class='big-column'>Variable</div> | Description |
| --- | --- |
| <textarea class="at-field" spellcheck="false" placeholder="deployment_key" id="deployment_key"></textarea> | (Required) The [deployment](../../general/data-model.md#deployments) key. |
| <textarea class="at-field" spellcheck="false" placeholder="user_id" id="user_id"></textarea> | (Required) The user ID used to fetch variants. This should be the same [user](../../general/data-model.md#users) you [track exposure](./track-exposure.md) for. |
| <select id="server-zone" onchange="updateUrl()"><option value="US">US</option><option value="EU">EU</option></select> | The server zone for your Amplitude project |
| <a class="md-button" id="at-action-button">Fetch Variants</a> | |

Result: <span id="failure_tip"></span>
<pre>
<code id="result">
</code>
</pre>

<script src="/javascripts/api-table.js">
</script>

<script>
document.getElementById('deployment_key').value =
     localStorage.getItem('deployment_key') || '';

function updateUrl() {
     const serverZone = document.getElementById('server-zone').value;
     const url = serverZone === 'US' ? 'https://api.lab.amplitude.com/v1/vardata?user_id=' :
          'https://api.lab.eu.amplitude.com/v1/vardata?user_id=';

     document.getElementById('curl_url').textContent = url;
}

document.getElementById('curl_url').textContent = 'https://api.lab.amplitude.com/v1/vardata?user_id=';

setupApiTable({
     'deployment_key': false,
     'user_id': true
}, async function(fields) {
     const deploymentKey = fields['deployment_key'];
     const userId = fields['user_id'];
     const serverZone = document.getElementById('server-zone').value;

     localStorage.setItem('deployment_key', deploymentKey);

     const url = serverZone === 'US' ? 'https://api.lab.amplitude.com/v1/vardata?user_id=' + userId :
          'https://api.lab.eu.amplitude.com/v1/vardata?user_id=' + userId;

     const response = await fetch(url, {
          headers: {
               'Authorization': 'Api-Key ' + deploymentKey,
          },
     });
     if (response.status != 200) {
          const body = await response.text();
          throw Error(response.status + ': ' + body);
     }
     const result = await response.json();
     return JSON.stringify(result, null, 2);
});
</script>

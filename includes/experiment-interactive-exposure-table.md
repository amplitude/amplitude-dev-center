<!-- markdown-link-check-disable -->
!!!warning "Potential data changes"
    This example makes real requests to the API and can potentially change the data in your Amplitude project. We recommend using a development project when testing APIs.

<pre>
<code>curl --request POST \
     --url '<span id="curl_url"></span>' \
     --data '{"api_key":"<span id="curl_api_key"></span>","events":[{"event_type":"$exposure","user_id":"<span id="curl_user_id"></span>","event_properties":{"flag_key":"<span id="curl_flag_key"></span>","variant":"<span id="curl_variant"></span>"}}]}'
</code>
</pre>

| <div class='big-column'>Variable</div> | Description |
| --- | --- |
|  <textarea class="at-field" spellcheck="false" placeholder="analytics_api_key" id="api_key">api_key</textarea> | The analytics API key from [project](../../general/data-model.md#projects) which you created your [flag](../../general/data-model.md#flags-and-experiments) and [deployment](../../general/data-model.md#deployments) in. |
| <textarea class="at-field" spellcheck="false" placeholder="user_id" id="user_id">user_id</textarea> | The user ID used to fetch variants. This should be the same [user](../../general/data-model.md#users) you [fetched variants](./fetch-variants.md) for. |
| <textarea class="at-field" spellcheck="false" placeholder="flag_key" id="flag_key"></textarea> | The flag key of the flag or experiment the user has been exposed to. |
| <textarea class="at-field" spellcheck="false" placeholder="variant" id="variant">on</textarea> | The variant key, `on` if you're using the default flag variant. |
| <select id="server-zone" onchange="updateUrl()"><option value="US">US</option><option value="EU">EU</option></select> | The server zone for your Amplitude project |
| <a class="md-button" id="at-action-button">Track Exposure</a> | |

Result: <span id="failure_tip"></span>
<pre>
<code id="result">
</code>
</pre>

<script src="/javascripts/api-table.js">
</script>

<script>
document.getElementById('api_key').value =
     localStorage.getItem('api_key') || '';

function updateUrl() {
     const serverZone = document.getElementById('server-zone').value;
     const url = serverZone === 'US' ? 'https://api2.amplitude.com/2/httpapi' :
          'https://api.eu.amplitude.com/2/httpapi';

     document.getElementById('curl_url').textContent = url;
}

document.getElementById('curl_url').textContent = 'https://api2.amplitude.com/2/httpapi';

setupApiTable({
     'api_key': false,
     'user_id': false,
     'flag_key': false,
     'variant': false,
}, async function(fields) {
     const apiKey = fields['api_key'];
     const userId = fields['user_id'];
     const flagKey = fields['flag_key'];
     const variant = fields['variant'];

     localStorage.setItem('api_key', apiKey);

     const serverZone = document.getElementById('server-zone').value;
     const url = serverZone === 'US' ? 'https://api2.amplitude.com/2/httpapi' :
          'https://api.eu.amplitude.com/2/httpapi';

     const response = await fetch(url, {
          method: 'POST',
          headers: {
               'Content-Type':'application/json',
               'Accept':'*/*'
          },
          body: JSON.stringify({
               "api_key": apiKey,
               "events":[{
                    "event_type":"$exposure",
                    "user_id": userId,
                    "event_properties":{
                         "flag_key": flagKey,
                         "variant": variant
                    }
               }]
          })
     });
     if (response.status != 200) {
          const body = await response.text();
          throw Error(response.status + ': ' + body);
     }
     const result = await response.json();
     return JSON.stringify(result, null, 2);
});
</script>
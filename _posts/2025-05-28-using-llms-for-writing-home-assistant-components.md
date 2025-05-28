---
title: "Using LLMs for writing Home Assistant components"
---

### Why I switched to Home Assistant?

I bought a Samsung SmartThings hub and various Zigbee 3.0 device when I was living in Germany. I bought the hub from UK. The problems started when I moved to Finland five years ago. Initially the system worked in Finland too, but I wasn't able to pair new sensors. Turned out my Samsung account was locked to the US region. I spent weeks talking with Samsung supports in the US, Germany, UK, and Finland. Someone even asked me to contact the Swedish support. Most representatives told me that it's not possible to change the account region, but someone told me that it would be possible, so I was persistent.

In the end I got my account moved to Finland, but that made things only worse. I contacted Samsung support again. They told me that I had reached the US support and I must contact their UK support because I have a UK account. The UK support told me that my data is in the US shard, and only the Finnish support can move it. I created several tickets in the Finnish support, but no one ever responded. Meanwhile I read about Home Assistant, ordered a Raspberry Pi, installed Home Assistant using [IOTstack](https://sensorsiot.github.io/IOTstack/), and never looked back.

### Reverse engineering the Hydrolink API

Until now, I haven't needed to develop any custom components for Home Assistant. But recently my condominium installed new water meters and I wanted to see if I could integrate them into Home Assistant.

Our new water meters send their readings periodically to the manufacturer's cloud. It's possible to check the readings by logging in to the cloud using a web browser. The login page is entirely generated using JavaScript. I haven't used JavaScript for ages. Naturally, the JavaScript code has been minified, making it impossible for me to understand a thing.

The first thing I did was running the JavaScript file through a code formatter. Since the minifier has replaces all the symbols with as short character sequences as possible, it still made very little sense to me. I wanted to see what I could make of it using LLMs. The file is huge, so I couldn't feed the whole file to an LLM. I had to find the interesting parts. My assumption was that the code authenticates to the manufacturer's cloud and receives a connection token. Searching for keywords such as "username" and "password", I found some promising sections.

```
ge = function (e, t, n) {
    return function (r) {
        return fe(void 0, void 0, void 0, (function () {
            return pe(this, (function (o) {
                return r(me(!0)), r({
                    type: he.LOGIN_ERROR,
                    payload: null
                }), fetch("".concat(getBaseUrl(), "/api/v2/login"), {
                    method: "POST",
                    headers: {
                        Accept: "application/json",
                        "Content-Type": "application/json",
                        Connection: "close"
                    },
                    body: JSON.stringify({
                        username: e,
                        password: t
                    })
                }).then((function (e) {
                    return e.json()
                })).catch((function (e) {
                    r(te("error", "JSON parsing error"))
                })).then((function (o) {
                    ...
                })).catch((function (e) {
                    r(me(!1)), r(te("error", "".concat(resolveModule("generic_login_error"), ": ").concat(e)))
                })), [2]
            }))
        }))
    }
},
```

Next I asked DeepSeek to replace the symbols with more descriptive names in the JavaScript code. The result started to look more readable already.

> Here's the refactored JavaScript code with more descriptive variable and function names:

```js
performLogin = function (username, password, rememberPassword) {
    return function (dispatch) {
        return handleAsync(void 0, void 0, void 0, (function () {
            return processGenerator(this, (function (generatorContext) {
                return dispatch(setLoadingState(true)),
                dispatch({
                    type: actionTypes.LOGIN_ERROR,
                    payload: null
                }),
                fetch(`${getBaseUrl()}/api/v2/login`, {
                    method: "POST",
                    headers: {
                        Accept: "application/json",
                        "Content-Type": "application/json",
                        Connection: "close"
                    },
                    body: JSON.stringify({
                        username: username,
                        password: password
                    })
                })
                .then((function (response) {
                    return response.json()
                }))
                .catch((function (error) {
                    dispatch(showNotification("error", "JSON parsing error"))
                }))
                .then((function (responseData) {
                    ...
                }))
                .catch((function (error) {
                    dispatch(setLoadingState(false)),
                    dispatch(showNotification("error", `${resolveModule("generic_login_error")}: ${error}`))
                })),
                [2]
            })
        }))
    }
}
```

Next I asked DeepSeek to write it in Python.

> Here's a complete Python script that logs in to the API, using the **requests** library (synchronous) for simplicity:

```python
...

def login(username: str, password: str, remember_password: bool = False) -> bool:
    """Perform login to the API"""
    headers = {
        'Accept': 'application/json',
        'Content-Type': 'application/json',
        'Connection': 'close'
    }

    payload = {
        'username': username,
        'password': password
    }

    try:
        response = requests.post(
            f"{BASE_URL}{LOGIN_ENDPOINT}",
            headers=headers,
            json=payload
        )
        response.raise_for_status()  # Raises exception for 4XX/5XX errors

        response_data = response.json()

        if not response_data or 'token' not in response_data:
            print("Login failed: Incorrect username or password")
            return False

        # Save credentials if login successful
        save_credentials(username, response_data['token'], remember_password, password)
        print("Login successful!")
        print(f"Token: {response_data['token']}")
        return True

    except requests.exceptions.RequestException as e:
        print(f"Login failed: {str(e)}")
        return False
    except json.JSONDecodeError:
        print("Login failed: Invalid response from server")
        return False

...
```

Using this method I was quickly able to produce a script that connects to the manufacturer's cloud and fetches the latest meter readings.

### Writing a custom component for Home Assistant

I don't have any experience in writing code for Home Assistant. To create the project structure I used Aaron Godfrey's [cookiecutter project template](https://github.com/boralyl/cookiecutter-homeassistant-component). His excellent [tutorial](https://aarongodfrey.dev/home%20automation/building_a_home_assistant_custom_component_part_1/) was also really helpful later on.

To get started with the code, I asked DeepSeek to write the sensor. As the sensor code is pretty much contained in one small file, this is something that's particularly easy for an LLM to digest. Still, given how limited the documentation for the Home Assistant programming interface is and how little discussion there is about issues related to writing custom components, I was surprised how good code it was able to produce. Apparently the source code of Home Assistant and all the public components is all that is needed to train an LLM to write modules that interface with it.

What DeepSeek produced was by no means a finished product—in fact there was still work left for a couple of days. But when you don't know where to start, LLMs can save a ton of time by writing a scratch for you. They were also particularly useful in writing unit tests that mock the API. Even though the unit tests didn't work out of the box, having a scratch can save a lot of time since getting started is often the hard part.

The final Home Assistant component is published on [this GitHub repository](https://github.com/senarvi/hydrolink-home-assistant).

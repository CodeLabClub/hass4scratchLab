# hass4scratchLab

基于 `homeassistant==0.69.1`


# 调整之处
1. `site-packages/homeassistant/components/remote/__init__.py`

```python
REMOTE_SERVICE_SEND_COMMAND_SCHEMA = REMOTE_SERVICE_SCHEMA.extend({
    # wwj: add  ir_data_scratch 
    vol.Optional('ir_data_scratch'): cv.string,
    vol.Required(ATTR_COMMAND): vol.All(cv.ensure_list, [cv.string]),
    vol.Optional(ATTR_DEVICE): cv.string,
    vol.Optional(
        ATTR_NUM_REPEATS, default=DEFAULT_NUM_REPEATS): cv.positive_int,
    vol.Optional(ATTR_DELAY_SECS): vol.Coerce(float),
})
```

2.  `site-packages/homeassistant/components/remote/xiaomi_miio.py`

```python
PLATFORM_SCHEMA = PLATFORM_SCHEMA.extend({
    # wwj: add  ir_data_scratch 
    vol.Optional('ir_data_scratch'): cv.string,
    vol.Optional(CONF_NAME): cv.string,
    vol.Required(CONF_HOST): cv.string,
    vol.Optional(CONF_TIMEOUT, default=DEFAULT_TIMEOUT):
        vol.All(int, vol.Range(min=0)),
    vol.Optional(CONF_SLOT, default=DEFAULT_SLOT):
        vol.All(int, vol.Range(min=1, max=1000000)),
    vol.Optional(ATTR_HIDDEN, default=True): cv.boolean,
    vol.Required(CONF_TOKEN): vol.All(str, vol.Length(min=32, max=32)),
    vol.Optional(CONF_COMMANDS, default={}):
        vol.Schema({cv.slug: COMMAND_SCHEMA}),
}, extra=vol.ALLOW_EXTRA)
```

```python
    def send_command(self, command, **kwargs):
        """Wrapper for _send_command."""
        num_repeats = kwargs.get(ATTR_NUM_REPEATS)

        delay = kwargs.get(ATTR_DELAY_SECS, DEFAULT_DELAY_SECS)
        ir_data_scratch = kwargs.get("ir_data_scratch")
        for _ in range(num_repeats):
            for payload in command:
                # command: dynamic_conmand # wwj
                if payload in self._commands:
                    self._send_command(ir_data_scratch)
                    # for local_payload in self._commands[payload][CONF_COMMAND]:
                        # import IPython;IPython.embed()
                        # print("local_payload:", local_payload)
                        # self._send_command(local_payload)
                else:
                    self._send_command(payload)
                time.sleep(delay)
```

```python
        while (utcnow() - start_time) < timedelta(seconds=timeout):
            message = yield from hass.async_add_job(
                device.read, slot)
            _LOGGER.debug("Message received from device: '%s'", message)

            if 'code' in message and message['code']:
                log_msg = "Received command is: {}".format(message['code'])
                _LOGGER.info(log_msg)
                hass.components.persistent_notification.async_create(
                    log_msg, title='Xiaomi Miio Remote')
                # emit a event
                hass.bus.fire('my_cool_event', { 'answer': 42 }) # Q: 未生效,使用persistent_notification也够
                print("test fire event")
                return
```

# Simulation in RoboCasa
I've had to set up <a href="https://robocasa.ai">Robocasa</a> to simulate manipulation tasks.

**IMAGE OR VIDEO HERE**

1. <a href="#installation">Installation</a>
2. <a href="#installation">Installation</a>

## Installation

To install RoboCasa, follow the comprehensive instructions provided in the [RoboCasa documentation](https://github.com/robocasa/robocasa).

### Setting Up the SpaceMouse Pro

To set up a SpaceMouse Pro, refer to the [Robosuite documentation](https://robosuite.ai/docs/source/robosuite.devices.html#module-robosuite.devices.spacemouse). You might need to update the `SPACEMOUSE_VENDOR_ID` and `SPACEMOUSE_PRODUCT_ID` in `robocasa/macros_private.py` to match your SpaceMouse. You can find these IDs using the command:

```bash
$ lsusb
```

For some SpaceMouse models, you may need to manually activate the logic by modifying the `run` method. Add `or True` to the method definition as shown below:

``` python
while True:
    d = self.device.read(13)
    if d is not None and self._enabled:

        if self.product_id == 50741 or True:
```


require('./Base');

const inherits = require('util').inherits;
const miio = require('miio');

var Accessory, PlatformAccessory, Service, Characteristic, UUIDGen;

ZhiMiFan2S = function(platform, config) {
    this.init(platform, config);
    
    Accessory = platform.Accessory;
    PlatformAccessory = platform.PlatformAccessory;
    Service = platform.Service;
    Characteristic = platform.Characteristic;
    UUIDGen = platform.UUIDGen;
    
    this.device = new miio.Device({
        address: this.config['ip'],
        token: this.config['token']
    });
    
    this.accessories = {};
    if(!this.config['fanDisable'] && this.config['fanName'] && this.config['fanName'] != "") {
        this.accessories['fanAccessory'] = new ZhiMiFWFanFanAccessory(this);
    }
    if(!this.config['buzzerSwitchDisable'] && this.config['buzzerSwitchName'] && this.config['buzzerSwitchName'] != "") {
        this.accessories['buzzerSwitchAccessory'] = new ZhiMiFWFanBuzzerSwitchAccessory(this);
    }
    if(!this.config['ledBulbDisable'] && this.config['ledBulbName'] && this.config['ledBulbName'] != "") {
        this.accessories['ledBulbAccessory'] = new ZhiMiFWFanLEDBulbAccessory(this);
    }
    if(!this.config['manualSwingDisable'] && this.config['manualSwingName']['right'] && this.config['manualSwingName']['right'] != "" && this.config['manualSwingName']['left'] && this.config['manualSwingName']['left'] != "") {
        this.accessories['manualSwingRightAccessory'] = new ZhiMiFWManualSwingAccessory(this, 'right');
        this.accessories['manualSwingLeftAccessory'] = new ZhiMiFWManualSwingAccessory(this, 'left');
    }
    var accessoriesArr = this.obj2array(this.accessories);
    return accessoriesArr;
}
inherits(ZhiMiFan2S, Base);

ZhiMiFWFanFanAccessory = function(dThis) {
    this.device = dThis.device;
    this.name = dThis.config['fanName'];
    this.config = dThis.config;
    this.platform = dThis.platform;
}

ZhiMiFWFanFanAccessory.prototype.getServices = function() {
    var that = this;
    var services = [];

    var infoService = new Service.AccessoryInformation();
    infoService
        .setCharacteristic(Characteristic.Manufacturer, "Xiaomi")
        .setCharacteristic(Characteristic.Model, "SmartMi Fan 2S")
        .setCharacteristic(Characteristic.SerialNumber, "Undefined");
    services.push(infoService);

    var fanService = new Service.Fanv2(this.name);
    var activeCharacteristic = fanService.getCharacteristic(Characteristic.Active);
    if (!this.config['fanLockPhysicalControlsSwitchDisable']) {
        var lockPhysicalControlsCharacteristic = fanService.addCharacteristic(Characteristic.LockPhysicalControls);
    }
    if (!this.config['fanSwingModeSwitchDisable']) {
        var swingModeControlsCharacteristic = fanService.addCharacteristic(Characteristic.SwingMode);
    }
    if (!this.config['fanRotationSpeedDisable']) {
        var rotationSpeedCharacteristic = fanService.addCharacteristic(Characteristic.RotationSpeed);
    }

    // power
    activeCharacteristic
        .on('get', function(callback) {
            that.device.call("get_prop", ["power"]).then(result => {
                callback(null, result[0] === "on" ? Characteristic.Active.ACTIVE : Characteristic.Active.INACTIVE);
            }).catch(function(err) {
                callback(err);
            });
        }.bind(this))
        .on('set', function(value, callback) {
            that.device.call("set_power", [value ? "on" : "off"]).then(result => {
                if(result[0] === "ok") {
                    callback(null);
                } else {
                    callback(new Error(result[0]));
                }
            }).catch(function(err) {
                callback(err);
            });
        }.bind(this));
    
    // child_lock
    if (!this.config['fanLockPhysicalControlsSwitchDisable']) {
    lockPhysicalControlsCharacteristic
        .on('get', function(callback) {
            that.device.call("get_prop", ["child_lock"]).then(result => {
                callback(null, result[0] === "on" ? Characteristic.LockPhysicalControls.CONTROL_LOCK_ENABLED : Characteristic.LockPhysicalControls.CONTROL_LOCK_DISABLED);
            }).catch(function(err) {
                callback(err);
            });
        }.bind(this))
        .on('set', function(value, callback) {
            that.device.call("set_child_lock", [value ? "on" : "off"]).then(result => {
                if(result[0] === "ok") {
                    callback(null);
                } else {
                    callback(new Error(result[0]));
                }
            }).catch(function(err) {
                callback(err);
            });
        }.bind(this));
    }

    // angle_enable
    if (!this.config['fanSwingModeSwitchDisable']) {
    swingModeControlsCharacteristic
        .on('get', function(callback) {
            that.device.call("get_prop", ["angle_enable"]).then(result => {
                callback(null, result[0] === "on" ? Characteristic.SwingMode.SWING_ENABLED : Characteristic.SwingMode.SWING_DISABLED);
            }).catch(function(err) {
                callback(err);
            });
        }.bind(this))
        .on('set', function(value, callback) {
            that.device.call("set_angle_enable", [value ? "on" : "off"]).then(result => {
                if(result[0] === "ok") {
                    callback(null);
                } else {
                    callback(new Error(result[0]));
                }
            }).catch(function(err) {
                callback(err);
            });
        }.bind(this));
    }

    // speed_level natural_level
    if (!this.config['fanRotationSpeedDisable']) {
    rotationSpeedCharacteristic
        .on('get', function(callback) {
            that.device.call("get_prop", ["speed_level"]).then(result => {
                callback(null, result[0]);
            }).catch(function(err) {
                callback(err);
            });
        }.bind(this))
        .on('set', function(value, callback) {
            if(value > 0) {
                that.device.call("set_speed_level", [value]).then(result => {
                    if(result[0] === "ok") {
                        callback(null);
                    } else {
                        callback(new Error(result[0]));
                    }
                }).catch(function(err) {
                    callback(err);
                });
            }
        }.bind(this));
    }

    services.push(fanService);
    return services;
}

ZhiMiFWFanBuzzerSwitchAccessory = function(dThis) {
    this.device = dThis.device;
    this.name = dThis.config['buzzerSwitchName'];
    this.platform = dThis.platform;
}

ZhiMiFWFanBuzzerSwitchAccessory.prototype.getServices = function() {
    var services = [];

    var infoService = new Service.AccessoryInformation();
    infoService
        .setCharacteristic(Characteristic.Manufacturer, "Xiaomi")
        .setCharacteristic(Characteristic.Model, "SmartMi Fan 2S")
        .setCharacteristic(Characteristic.SerialNumber, "Undefined");
    services.push(infoService);
    
    var switchService = new Service.Switch(this.name);
    switchService
        .getCharacteristic(Characteristic.On)
        .on('get', this.getBuzzerState.bind(this))
        .on('set', this.setBuzzerState.bind(this));
    services.push(switchService);

    return services;
}

ZhiMiFWFanBuzzerSwitchAccessory.prototype.getBuzzerState = function(callback) {
    var that = this;
    this.device.call("get_prop", ["buzzer"]).then(result => {
        callback(null, result[0] === "on" ? 1 : 0);
    }).catch(function(err) {
        callback(err);
    });
}

ZhiMiFWFanBuzzerSwitchAccessory.prototype.setBuzzerState = function(value, callback) {
    var that = this;
    that.device.call("set_buzzer", [value ? "on" : "off"]).then(result => {
        if(result[0] === "ok") {
            callback(null);
        } else {
            callback(new Error(result[0]));
        }
    }).catch(function(err) {
        callback(err);
    });
}

ZhiMiFWManualSwingAccessory = function(dThis, direction) {
    this.device = dThis.device;
    this.name = dThis.config['manualSwingName'][direction];
    this.platform = dThis.platform;
    this.direction = direction;
}

ZhiMiFWManualSwingAccessory.prototype.getServices = function() {
    var services = [];
    var manualSwingService = new Service.Switch(this.name);
    manualSwingService
        .getCharacteristic(Characteristic.On)
        .on('get', this.getManualSwingState.bind(this))
        .on('set', this.setManualSwingState.bind(this));
    services.push(manualSwingService);
    return services;
}

ZhiMiFWManualSwingAccessory.prototype.getManualSwingState = function(callback) {
    callback(null, 0);
}

ZhiMiFWManualSwingAccessory.prototype.setManualSwingState = function(value, callback) {
    var that = this;
    if (value) {
        // first check if the fan is enabled and auto swing is disabled
        this.device.call("get_prop", ["power", "angle_enable"]).then(result => {
            if ((result[0] === "on") && (result[1] === "off")) {
                that.device.call("set_move", [that.direction]).then(result => {
                    if(result[0] === "ok") {
                        callback(null);
                    } else {
                        callback(new Error(result[0]));
                    }
                }).catch(function(err) {
                    callback(err);
                });
            } else {
                callback(null);
            }
        }).catch(function(err) {
            callback(err);
        });
    } else {
        callback(null);
    }
}

ZhiMiFWFanLEDBulbAccessory = function(dThis) {
    this.device = dThis.device;
    this.name = dThis.config['ledBulbName'];
    this.platform = dThis.platform;
}

ZhiMiFWFanLEDBulbAccessory.prototype.getServices = function() {
    var that = this;
    var services = [];

    var infoService = new Service.AccessoryInformation();
    infoService
        .setCharacteristic(Characteristic.Manufacturer, "Xiaomi")
        .setCharacteristic(Characteristic.Model, "SmartMi Fan 2S")
        .setCharacteristic(Characteristic.SerialNumber, "Undefined");
    services.push(infoService);

    var switchLEDService = new Service.Lightbulb(this.name);
    var onCharacteristic = switchLEDService.getCharacteristic(Characteristic.On);
    var brightnessCharacteristic = switchLEDService.addCharacteristic(Characteristic.Brightness);

    onCharacteristic
        .on('get', function(callback) {
            this.device.call("get_prop", ["led_b"]).then(result => {
                callback(null, result[0] === 2 ? false : true);
            }).catch(function(err) {
                callback(err);
            });
        }.bind(this))
        .on('set', function(value, callback) {
            that.setLedB(value ? that.getLevelByBrightness(brightnessCharacteristic.value) : 2, callback);
        }.bind(this));
    brightnessCharacteristic
        .on('get', function(callback) {
            this.device.call("get_prop", ["led_b"]).then(result => {
                if(result[0] == 0) {
                    if(brightnessCharacteristic.value > 50 && brightnessCharacteristic.value <= 100) {
                        callback(null, brightnessCharacteristic.value);
                    } else {
                        callback(null, 100);
                    }
                } else if(result[0] == 1) {
                    if(brightnessCharacteristic.value > 0 && brightnessCharacteristic.value <= 50) {
                        callback(null, brightnessCharacteristic.value);
                    } else {
                        callback(null, 50);
                    }
                } else if(result[0] == 2) {
                    callback(null, 0);
                }
            }).catch(function(err) {
                callback(err);
            });
        }.bind(this));
    services.push(switchLEDService);

    return services;
}

ZhiMiFWFanLEDBulbAccessory.prototype.setLedB = function(led_b, callback) {
    var that = this;
    this.device.call("set_led_b", [led_b]).then(result => {
        if(result[0] === "ok") {
            callback(null);
        } else {
            callback(new Error(result[0]));
        }
    }).catch(function(err) {
        callback(err);
    });
}

ZhiMiFWFanLEDBulbAccessory.prototype.getLevelByBrightness = function(brightness) {
    if(brightness == 0) {
        return 2;
    } else if(brightness > 0 && brightness <= 50) {
        return 1;
    } else if (brightness > 50 && brightness <= 100) {
        return 0;
    }
}

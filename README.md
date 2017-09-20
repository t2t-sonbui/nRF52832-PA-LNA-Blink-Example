# nRF52832-PA/LNA Blink Fix not work!
===================
Test with nRF5_SDK_14.0.0_3bcc1f7 Soft132 v5.0
-------------
Add define **APP_PA_LAN**
-------------

**//sdk_config.h**
Add define:
```ruby
#ifndef NRF_SDH_CLOCK_LF_XTAL_ACCURACY_PA
#define NRF_SDH_CLOCK_LF_XTAL_ACCURACY_PA 0
#endif
```

**//nf_sdh.c**
Edit
```
ret_code_t nrf_sdh_enable_request(void)
{
...
nrf_clock_lf_cfg_t const clock_lf_cfg =
#ifdef APP_PA_LAN
    {   .source        = NRF_CLOCK_LF_SRC_SYNTH,
        .rc_ctiv       = NRF_SDH_CLOCK_LF_RC_CTIV,
        .rc_temp_ctiv  = NRF_SDH_CLOCK_LF_RC_TEMP_CTIV,
        .accuracy      = NRF_SDH_CLOCK_LF_XTAL_ACCURACY_PA
    };
#else
        {
            .source        = NRF_SDH_CLOCK_LF_SRC,
            .rc_ctiv       = NRF_SDH_CLOCK_LF_RC_CTIV,
            .rc_temp_ctiv  = NRF_SDH_CLOCK_LF_RC_TEMP_CTIV,
#ifdef S132
            .accuracy      = NRF_SDH_CLOCK_LF_XTAL_ACCURACY
#else
            .xtal_accuracy = NRF_SDH_CLOCK_LF_XTAL_ACCURACY
#endif
        };
#endif
...
}
```
**//main.c**
Add config the PA/LNA before main funciton
```
#define APP_PA_LAN
static void pa_lna_assist(uint32_t gpio_pa_pin, uint32_t gpio_lna_pin)
{
    ret_code_t err_code;
    nrf_gpio_cfg_output(gpio_pa_pin);
    nrf_gpio_pin_clear(gpio_pa_pin); //
    nrf_gpio_cfg_output(gpio_lna_pin);
    nrf_gpio_pin_clear(gpio_lna_pin); //
    static const uint32_t gpio_toggle_ch = 0;
    static const uint32_t ppi_set_ch = 0;
    static const uint32_t ppi_clr_ch = 1;

    // Configure SoftDevice PA/LNA assist
    static ble_opt_t opt;
    memset(&opt, 0, sizeof(ble_opt_t));
    // Common PA/LNA config
    opt.common_opt.pa_lna.gpiote_ch_id  = gpio_toggle_ch;        // GPIOTE channel
    opt.common_opt.pa_lna.ppi_ch_id_clr = ppi_clr_ch;            // PPI channel for pin clearing
    opt.common_opt.pa_lna.ppi_ch_id_set = ppi_set_ch;            // PPI channel for pin setting
    // PA config
    opt.common_opt.pa_lna.pa_cfg.active_high = 1;                // Set the pin to be active high
    opt.common_opt.pa_lna.pa_cfg.enable      = 1;                // Enable toggling
    opt.common_opt.pa_lna.pa_cfg.gpio_pin    = gpio_pa_pin;      // The GPIO pin to toggle

    // LNA config
    opt.common_opt.pa_lna.lna_cfg.active_high  = 1;              // Set the pin to be active high
    opt.common_opt.pa_lna.lna_cfg.enable       = 1;              // Enable toggling
    opt.common_opt.pa_lna.lna_cfg.gpio_pin     = gpio_lna_pin;   // The GPIO pin to toggle

    err_code = sd_ble_opt_set(BLE_COMMON_OPT_PA_LNA, &opt);
    APP_ERROR_CHECK(err_code);
}
#endif
```

**Main funciton:**
Add after funciton "ble_stack_init()"
```
#ifdef APP_PA_LAN
    pa_lna_assist(24,20);//Pin 0.24 and pin 0.20
#endif
```

### Support 


# ZMK nrf52833

Instrucciones de como configurar ZMK para que funcione con nrf52833

Veamos primero algunas diferencias entre E73 2G4M08SIE (nrf52833) y E73 2G4M08S1C (nrf52840)

| | E73 2G4M08S1C (nrf52840) | E73 2G4M08SIE (nrf52833) |
|---|---|---|
| Memoria | 1024 kB | 512 kB |
| RAM | 256 kB | 128 kB |
| Pin 1 | P1.11 | No conectado |
| Pin 25 | DCCH | No conectado |
| DC/DC output | Si | No |
| SD | Si | Memoria insuficiente |

| Particiones | nrf52840 | nrf52833 |
|---|---|---|
| SD/MBR partición | 152kB | 4kB |
| CODE partición | 792kB | 428kB |
| Storage partición | 32kB | 32kB |
| Boot partición | 48kB | 48kB |

Dado que con nrf52833 solo tenemos 428kB disponibles no será posible tener una pantalla, o de tener una debe ser con un código muy limitado, el código básico de zmk para un teclado 60% con dos capas funcionando por usb y bluetooth necesita 312kB, si a eso sumamos la funcionalidad de rgb underglow sube a 328kB

Dicho esto veamos que configuraciones hay que hacer, estos ejemplos son en base a una board arm como nice_nano o nrfmicro

1. Cambiar la configuración SOC en el archivo **_config.c**

~~~
CONFIG_SOC_NRF52833_QIAA=y
~~~

2. Cambiar importancion dtsi base y definición de particiones en el archivo **.dts**

~~~
#include <nordic/nrf52833_qiaa.dtsi>

&flash0 {
  partitions {
    compatible = "fixed-partitions";
    #address-cells = <1>;
    #size-cells = <1>;

    sd_partition: partition@0 {
      label = "mbr";
      reg = <0x00000000 0x00001000>;
    };

    code_partition: partition@1000 {
      label = "code_partition";
      reg = <0x00001000 0x0006b000>;
    };

    storage_partition: partition@6c000 {
      label = "storage";
      reg = <0x0006c000 0x00008000>;
    };

    boot_partition: partition@74000 {
      label = "adafruit_boot";
      reg = <0x00074000 0x0000c000>;
    };
  };
};
~~~

3. Cambiar en el archivo **CMakeLists.txt** la posicion donde empieza a escribir el código

~~~
  -b 0x1000
~~~
 
Con estos cambios ya podremos disfrutar de ZMK en nuestro chip nrf52833

Adicional desde este repo puedes descargar el **bootloader** correcto con nosd para este chip

##nrf52833 también es llamado como pca10100 y el bootloader debe ser nosd

El bootloader proviene del repo de adafruit, para flashear el micro debes pulsar dos veces reset muy rápido eso hará que aparezca como un pendrive, entonces copias el archivo .uf2 generado por ZMK.

##Otros links

[ZMK Firmware](https://zmk.dev/)

[Adafruit nr52 bootloader releases](https://github.com/adafruit/Adafruit_nRF52_Bootloader/releases)

[Open source VIA Español todos los teclados QMK soportados](https://github.com/ci-bus/Miguelio-VIA-Keyboards)

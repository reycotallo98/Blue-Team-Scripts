# Integración de Wazuh con OpenCTI

Este proyecto documenta cómo integrar **Wazuh** con **OpenCTI** mediante dos scripts personalizados para enviar alertas desde Wazuh hacia la plataforma de inteligencia de amenazas OpenCTI. 
Esta integración permite enriquecer la información de seguridad en tiempo real y aprovechar las capacidades analíticas de OpenCTI.

## Requisitos

Antes de comenzar, asegúrate de contar con los siguientes requisitos:

- Instancia funcional de **Wazuh** (v4.x o superior)
- Instancia funcional de **OpenCTI** (v5.2 o superior)
- Acceso API a OpenCTI con un token válido
- Conexión de red entre ambos sistemas

## Configuración

### Wazuh
#### Ossec
En el archivo ossec.conf (/var/ossec/etc/ossec.conf), añadiremos lo siguiente al final del archivo, sustituyendo API_KEY y URL_OPENCTI:

```xml
<ossec_config>
  <integration>
     <name>custom-opencti</name>
  <group>sysmon_event1,sysmon_event3,sysmon_event6,sysmon_event7,sysmon_event_15,sysmon_event_22,syscheck</group>
     <alert_format>json</alert_format>
     <api_key>API_KEY</api_key>
     <hook_url>URL_OPENCTI/graphql</hook_url>
  </integration>
</ossec_config>
```
#### Scripts
Añadimos los archivos descargables en este repositorio en el directorio de integraciones (/var/ossec/integrations), manteniendo sus nombres, y le damos los permisos necesarios con el siguiente comando:
```bash
chmod 755 custom-opencti*
```
#### Reglas
Añadir las siguientes reglas a Wazuh:
```xml
<group name="threat_intel,">
 <rule id="100623" level="10">
    <field name="integration">opencti</field>
    <description>OpenCTI</description>
    <group>opencti,</group>
    <options>no_full_log</options>
  </rule>
<rule id="100624" level="5">
    <if_sid>100623</if_sid>
    <field name="opencti.error">\.+</field>
    <description>OpenCTI - Error connecting to API</description>
    <options>no_full_log</options>
    <group>opencti,opencti_error,</group>
  </rule>
<rule id="100625" level="12">
    <if_sid>100623</if_sid>
    <field name="opencti.id">\.+</field>
    <description>OpenCTI - IoC found in Threat Intel - $(opencti.observable_value)</description>
    <options>no_full_log</options>
    <group>opencti,opencti_alert,</group>
  </rule>
</group>
```
#### Reiniciar Wazuh Manager
Por último reiniciamos Wazuh:
```bash
sudo systemctl restart wazuh-manager
```

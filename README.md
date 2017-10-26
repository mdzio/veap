# Very Easy Automation Protocol (VEAP)

Im Folgenden wird ein TCP/IP-basiertes Kommunikationsprotokoll für den Bereich Automatisierung vorgestellt, das **einfach verständlich** und **leicht zu implementieren** ist und dennoch auch weitergehende Anforderungen, wie die **Modellierung von Datenmodellen** oder 
**Sicherheitsanforderungen**, erfüllt.

Das Protokoll kann beispielsweise im Bereich der Gebäudeautomatisierung oder für das Internet-der-Dinge eingesetzt werden. Es kann Geräte untereinander, mit überlagerten Systemen (z.B. Zentralen, Betriebsdatenerfassung) oder mit grafischen Benutzerschnittstellen verbinden.

Das VEAP-Protokoll befindet sich zurzeit noch in der Entwicklung!

### Begründung für ein neues Protokoll

 Bereich der Automatisierung und dem Internet-der-Dinge existiert bereits eine Vielzahl an herstellerunabhängigen Protokollen (u.a. [MQTT](https://de.wikipedia.org/wiki/MQTT), [CoAP](https://de.wikipedia.org/wiki/Constrained_Application_Protocol), [OPC-UA](https://de.wikipedia.org/wiki/OPC_Unified_Architecture)). Eine Untersuchung hat allerdings ergeben, dass keines dieser Kommunikationsprotokolle alle oben geforderten Eigenschaften besitzt. Sie sind beispielsweise funktional eingeschränkt (z.B. MQTT) oder aufwändig zu implementieren (z.B. CoAP, OPC-UA). 

**Es gibt natürlich andere Anforderungen, die nur von einem der o.g. Protokolle (MQTT, CoAP oder OPC-UA) erfüllt werden und nicht von VEAP. Jedes Protokoll besitzt eigene Herausstellungsmerkmale.**

## Lizenz

Dieses Werk ist lizenziert unter einer [Creative Commons Namensnennung - Weitergabe unter gleichen Bedingungen 4.0 International Lizenz](http://creativecommons.org/licenses/by-sa/4.0/).

[![Lizenz](https://i.creativecommons.org/l/by-sa/4.0/88x31.png)](https://creativecommons.org/licenses/by-sa/4.0/)


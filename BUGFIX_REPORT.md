# Bugfix Report: LWJGL Kernel32 Crash

## Problembeschreibung
Beim Starten von Minecraft 1.20.1 mit der Mod stürzte das Spiel mit folgendem Fehler ab:
```
java.lang.NoClassDefFoundError: org/lwjgl/system/windows/Kernel32
```

### Ursache
Die Klasse `me.cortex.voxy.common.util.ThreadUtils` importierte direkt `org.lwjgl.system.windows.Kernel32`.
In der Laufzeitumgebung von Minecraft 1.20.1 (Fabric) ist diese spezifische Klasse der LWJGL-Bibliothek entweder nicht vorhanden, nicht im Classpath oder wurde relocated/gestripped. Der direkte Import führte daher zu einem Absturz, sobald die Klasse `ThreadUtils` geladen wurde (noch vor der Ausführung von Code).

## Lösung
Die Datei `src/main/java/me/cortex/voxy/common/util/ThreadUtils.java` wurde modifiziert:

1.  **Entfernung des Imports:** Der Import `import org.lwjgl.system.windows.Kernel32;` wurde entfernt.
2.  **Dynamisches Laden (Reflection-Alternative):** Anstatt die Klasse hardcodiert aufzurufen, wird nun versucht, die native Bibliothek `kernel32` dynamisch über LWJGLs `Library.loadNative` zu laden, um an die Funktionsadressen (`SetThreadPriority`, `SetThreadSelectedCpuSetMasks`) zu gelangen.
3.  **Fehlerbehandlung:** Falls die Bibliothek oder Funktionen nicht gefunden werden (was in dieser Umgebung der Fall zu sein scheint), fängt der Code den Fehler ab und deaktiviert die Windows-spezifischen Thread-Optimierungen, anstatt das Spiel abstürzen zu lassen.

## Status
Der Build ist nun stabil und das Spiel startet erfolgreich.

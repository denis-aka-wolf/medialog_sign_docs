---
title: Интеграция с КриптоПро
tags: [cryptopro, bridge, cades]
---

# CryptoPro Bridge

## Назначение

Bridge изолирует Windows COM automation от Flutter. Реализация находится в каталоге `medialog_sign_cryptopro/windows_bridge`, Dart-клиент в `flutter/lib/integrations/cryptopro/cryptopro_bridge_client.dart`.

## Стек

- Windows 10/11 x64.
- C# 7.3.
- .NET Framework 4.8.
- COM automation CAdESCOM.
- CryptoPro CSP и зарегистрированный x64 CAdESCOM.
- Flutter/Dart для вызывающей стороны.

## Протокол

Один процесс на один запрос. Запрос записывается в stdin как JSON; ответ читается из stdout как JSON-конверт.

Пример проверки доступности:

```json
{"action":"status"}
```

Пример списка сертификатов:

```json
{"action":"list-certificates"}
```

Dart-клиент реализует следующие действия:

| action | Назначение |
|---|---|
| `status` | Проверка COM-компонентов |
| `list-certificates` | Чтение `CurrentUser\\My` |
| `sign-detached` | Создание отсоединённой CAdES-BES |
| `sign-attached` | Создание присоединённой CAdES-BES |
| `verify-detached` | Проверка отсоединённой подписи |

Поля запроса подписания: `inputPath`, `outputPath`, `thumbprint`. Проверка использует `inputPath` и `signaturePath`.

## Модель сертификата в Dart

Класс `CryptoProCertificate` содержит:

- `thumbprint`
- `subjectName`
- `issuerName`
- `serialNumber`
- `validFrom`, `validTo`
- `hasPrivateKey`
- `dateRangeValid`
- `isReadyForSigning`

## Ошибки клиента

Клиент формирует `CryptoProBridgeException` с кодом и сообщением. В коде явно предусмотрены:
- `BRIDGE_NOT_FOUND`
- `BRIDGE_START_FAILED`
- `INVALID_BRIDGE_RESPONSE`
- `OPERATION_FAILED`
- `OPERATION_TIMEOUT`

Таймаут клиента по умолчанию: 90 секунд.

## Ограничения текущего каркаса

- Читается только `CurrentUser\\My` текущего Windows-профиля.
- Нет автоматического импорта сертификатов/контейнеров.
- Нет экспорта закрытых ключей.
- Нет автоматической проверки OCSP/CRL и политики доверия.
- Нет прямой записи в MSIGN.
- Совместимость с конкретным контуром НетрИки ещё должна быть подтверждена.
- COM-вызовы не были выполнены в среде разработки документации. Нужен тест на целевой Windows-машине.

## Сборка

Откройте `MedialogSign.CryptoProBridge.csproj` в Visual Studio 2022, выберите `Release | x64` и соберите. Ожидаемый путь:

`windows_bridge/bin/Release/MedialogSign.CryptoProBridge.exe`

Полный план: [[05-Сборка-и-запуск]].

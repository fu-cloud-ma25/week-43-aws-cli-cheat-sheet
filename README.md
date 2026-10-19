# Vecka 43: AWS CLI Cheat Sheet

AWS CLI (**AWS Command Line Interface**) är ett verktyg som gör det möjligt att kommunicera med AWS direkt från terminalen.

I kursen kommer vi framför allt använda AWS CLI för att konfigurera åtkomst till vårt AWS-konto. När detta är gjort kommer **Serverless Framework** kunna använda konfigurationen för att skapa och deploya resurser till AWS.

---

## Installation

Börja med att installera AWS CLI genom att följa AWS officiella installationsguide:

https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html

Välj det operativsystem du använder och följ instruktionerna.

När installationen är klar kan du kontrollera att AWS CLI fungerar genom att köra:

```bash
aws --version
```

Du bör få tillbaka information om vilken version av AWS CLI som är installerad.

Exempel:

```text
aws-cli/2.x.x Python/3.x.x ...
```

---

## Konfigurera AWS CLI

För att AWS CLI och Serverless Framework ska kunna kommunicera med ditt AWS-konto behöver du autentisera dig.

I den här kursen gör vi detta med hjälp av en **Access Key** som tillhör din IAM-användare.

Kör:

```bash
aws configure
```

Du kommer nu få ange följande information:

```text
AWS Access Key ID: <din access key>
AWS Secret Access Key: <din secret access key>
Default region name: eu-north-1
Default output format: json
```

Din **Access Key ID** och **Secret Access Key** hittar du/skapar du för din IAM-användare i AWS.

Vi använder regionen:

```text
eu-north-1
```

vilket är AWS-regionen **Europe (Stockholm)**.

---

## Viktigt om dina Access Keys

> ⚠️ **Din Secret Access Key är en hemlighet och ska behandlas på samma sätt som ett lösenord.**

Lägg aldrig dina AWS-nycklar i:

* GitHub
* JavaScript-filer
* `serverless.yml`
* README-filer
* filer som committas till Git
* Discord, Teams eller andra chattar

När du använder `aws configure` sparar AWS CLI informationen lokalt på din dator.

Du behöver därför **inte** lägga dina AWS-nycklar i projektet.

---

## Kontrollera att du är inloggad

När konfigurationen är klar kan du kontrollera att AWS CLI kan kommunicera med ditt AWS-konto:

```bash
aws sts get-caller-identity
```

Om allt fungerar bör du få tillbaka något liknande:

```json
{
  "UserId": "XXXXXXXXXXXX",
  "Account": "123456789012",
  "Arn": "arn:aws:iam::123456789012:user/student"
}
```

Det betyder att:

1. AWS CLI är installerat.
2. Dina credentials fungerar.
3. Du kan kommunicera med AWS.
4. AWS vet vilken användare du är.

---

## Kontrollera din konfiguration

Du kan kontrollera vilken konfiguration AWS CLI använder med:

```bash
aws configure list
```

Du kan då bland annat se vilken region och vilka credentials som används.

Exempel:

```text
      Name                    Value             Type
      ----                    -----             ----
   profile                <not set>             None
access_key     ****************ABCD shared-credentials-file
secret_key     ****************1234 shared-credentials-file
    region               eu-north-1      config-file
```

AWS visar inte hela dina nycklar.

---

## Arbeta med flera AWS-användare

Ibland kan du behöva arbeta med flera olika AWS-användare eller AWS-konton på samma dator. Istället för att köra `aws configure` varje gång och skriva över dina tidigare credentials kan du skapa olika **profiler**.

En profil innehåller credentials och konfiguration för en specifik AWS-användare.

### Skapa en ny profil

För att skapa en profil använder du `--profile` följt av det namn du vill ge profilen.

Exempel:

```bash id="m5z04d"
aws configure --profile school
```

Du får sedan ange credentials och region precis som tidigare:

```text id="tq3mlz"
AWS Access Key ID: <din access key>
AWS Secret Access Key: <din secret access key>
Default region name: eu-north-1
Default output format: json
```

Du kan exempelvis ha flera profiler:

```text id="5u8utv"
default
school
personal
```

På så sätt behöver du inte skriva över dina credentials när du vill använda en annan AWS-användare.

---

### Använd en specifik profil

När du kör ett AWS CLI-kommando kan du välja vilken profil som ska användas med `--profile`.

Exempel:

```bash id="nm2g9v"
aws sts get-caller-identity --profile school
```

eller:

```bash id="nl0yn1"
aws sts get-caller-identity --profile personal
```

Du kan då kontrollera att kommandot körs mot rätt användare och AWS-konto.

Om du **inte** anger någon profil:

```bash id="if54oz"
aws sts get-caller-identity
```

används profilen:

```text id="uwwy67"
default
```

---

### Byt profil i terminalen

Om du ska köra många kommandon med samma profil kan det bli jobbigt att skriva:

```text id="zhxx46"
--profile school
```

efter varje kommando.

Då kan du istället välja vilken profil terminalen ska använda.

#### PowerShell

```powershell id="3mbcsc"
$env:AWS_PROFILE="school"
```

#### Mac/Linux

```bash id="msbx2x"
export AWS_PROFILE=school
```

Efter detta kommer AWS CLI automatiskt använda profilen `school`.

Du kan kontrollera vilken användare som används:

```bash id="gn9ewh"
aws sts get-caller-identity
```

Vill du sedan byta till exempelvis din privata profil:

**PowerShell:**

```powershell id="u6kwc1"
$env:AWS_PROFILE="personal"
```

**Mac/Linux:**

```bash id="oc3m19"
export AWS_PROFILE=personal
```

> **OBS!** Detta byter bara profil för den aktuella terminalsessionen. Öppnar du en ny terminal används normalt din `default`-profil igen.

---

### Exempel

Om du har två AWS-användare kan du exempelvis konfigurera dem så här:

```bash id="5q1apc"
aws configure --profile school
aws configure --profile personal
```

Sedan kan du enkelt kontrollera dem:

```bash id="c7mpoo"
aws sts get-caller-identity --profile school
```

```bash id="1s45t6"
aws sts get-caller-identity --profile personal
```

Det är särskilt viktigt att kontrollera vilken profil du använder **innan du deployar resurser**, så att du inte råkar skapa resurser på fel AWS-konto.

---

# AWS CLI + Serverless Framework

Vi kommer **inte** att använda AWS CLI för att manuellt skapa och uppdatera våra Lambda-funktioner.

Istället kommer vi använda **Serverless Framework**.

AWS CLI sköter autentiseringen:

```text
Din dator
    │
    ▼
AWS CLI credentials
    │
    ▼
Serverless Framework
    │
    ▼
AWS
```

När du senare kör exempelvis:

```bash
serverless deploy
```

kan Serverless Framework använda dina lokala AWS-credentials för att autentisera sig mot AWS.

Serverless Framework kan därefter skapa och konfigurera de AWS-resurser som beskrivs i projektet.

Det innebär att vi slipper göra saker som att:

* manuellt skapa Lambda-funktioner
* manuellt skapa `.zip`-filer
* manuellt ladda upp vår kod
* skriva långa AWS CLI-kommandon för varje deployment

Istället beskriver vi vår serverless-applikation i kod och konfiguration och låter Serverless Framework hantera deploymenten.

---

## Kommandon att komma ihåg

### Kontrollera installationen

```bash
aws --version
```

### Konfigurera AWS

```bash
aws configure
```

### Kontrollera vilken användare du är

```bash
aws sts get-caller-identity
```

### Visa aktuell konfiguration

```bash
aws configure list
```

---

## Kort sammanfattning

AWS CLI gör det möjligt att kommunicera med AWS från terminalen.

I den här kursen använder vi framför allt AWS CLI för att:

1. Autentisera vår dator mot AWS.
2. Konfigurera vilken AWS-region vi arbetar mot.
3. Ge Serverless Framework möjlighet att kommunicera med AWS.

När AWS CLI är installerat och konfigurerat går vi vidare till **Serverless Framework**, där vi kommer skapa och deploya våra serverless-applikationer.

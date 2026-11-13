import os, zipfile, textwrap

root = "/mnt/data/HambaiTaxiAndroid"
workflow = os.path.join(root, ".github", "workflows", "android-apk.yml")
os.makedirs(os.path.dirname(workflow), exist_ok=True)
with open(workflow, "w", encoding="utf-8") as f:
    f.write(textwrap.dedent("""\
    name: Build HAMBAI TÁXI APK

    on:
      workflow_dispatch:
      push:
        branches: [ "main" ]

    jobs:
      build:
        runs-on: ubuntu-latest
        steps:
          - name: Checkout
            uses: actions/checkout@v4

          - name: Set up JDK 17
            uses: actions/setup-java@v4
            with:
              distribution: temurin
              java-version: "17"

          - name: Set up Gradle
            uses: gradle/actions/setup-gradle@v4
            with:
              gradle-version: "8.9"

          - name: Build debug APK
            run: gradle --no-daemon assembleDebug

          - name: Upload APK
            uses: actions/upload-artifact@v4
            with:
              name: hambai-taxi-debug-apk
              path: app/build/outputs/apk/debug/app-debug.apk
              if-no-files-found: error
    """))

readme_path = os.path.join(root, "README.md")
with open(readme_path, "a", encoding="utf-8") as f:
    f.write(textwrap.dedent("""

    ## Gerar o APK usando apenas o telemóvel (GitHub Actions)
    1. Crie um repositório GitHub e carregue todos os ficheiros desta pasta, mantendo as pastas.
    2. Confirme que o branch principal se chama `main`.
    3. No repositório, abra **Actions** e selecione **Build HAMBAI TÁXI APK**.
    4. Toque em **Run workflow** e confirme.
    5. Quando terminar com sucesso, abra a execução e descarregue o artefacto `hambai-taxi-debug-apk`.
    6. Extraia o ZIP do artefacto e instale `app-debug.apk` no Android. Poderá ser necessário autorizar a instalação desta origem nas definições do telefone.

    Se o GitHub indicar erro, envie uma captura de ecrã para corrigirmos. O APK gerado é de demonstração e não deve ser usado para viagens reais.
    """))

zip_path = "/mnt/data/HAMBAI_TAXI_Android_Prototype_v2.zip"
with zipfile.ZipFile(zip_path, "w", zipfile.ZIP_DEFLATED) as z:
    for folder, _, filenames in os.walk(root):
        for filename in filenames:
            full = os.path.join(folder, filename)
            z.write(full, os.path.relpath(full, "/mnt/data"))

print(zip_path)

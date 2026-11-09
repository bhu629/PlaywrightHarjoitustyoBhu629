# Järjestelmätesti Playwrightilla – Pants-pöytävarausjärjestelmä

## Tavoite

Pants on ravintolan pöytävarausjärjestelmä. Projektin aiemmat Playwright-testit
korvasivat backendin vastaukset itse keksityillä vastauksilla (mockeilla), joten
ne testasivat vain käyttöliittymää.

Harjoitustyön tavoitteena oli tehdä järjestelmätesti, joka testaa sovellusta
kokonaisuutena ilman mockeja: oikea selain, oikea frontend, oikea backend ja
tietokanta. Testattavaksi toiminnoksi valitsin kirjautumisen, koska ilman sitä
ravintolan omistaja ei pääse hallintanäkymään.

## Teknologiat

- **Playwright (TypeScript):** ohjaa selainta ja tarkistaa tulokset
- **React + Vite:** sovelluksen frontend
- **Spring Boot:** sovelluksen backend, joka tarkistaa tunnukset
- **H2-tietokanta:** testitietokanta, jossa on valmiina testikäyttäjä

## Toteutus

Testi on tiedostossa `frontend/tests/system-login.spec.ts`:

    test('admin can login with the correct backend', async ({ page }) => {
      await page.goto('/login');

      await page.getByLabel('Email').fill('admin@test.com');
      await page.getByLabel('Password').fill('admin123');
      await page.getByRole('button', { name: 'Sign in' }).click();

      await expect(page).toHaveURL(/\/owner\/dashboard$/);
      await expect(page.getByRole('heading', { name: 'Reservations' })).toBeVisible();
    });

### Mitä testissä tapahtuu

1. Selain avaa kirjautumissivun.
2. Testi täyttää sähköpostin ja salasanan ja painaa *Sign in* -painiketta.
3. Frontend lähettää tunnukset backendille. Backend tarkistaa ne tietokannasta ja
   palauttaa kirjautumistokenin.
4. Testi tarkistaa, että käyttäjä päätyy dashboardille ja näkee otsikon
   *Reservations*.

### Ero aiempiin testeihin

Aiemmassa testissä `login.spec.ts` Playwright vastaa kirjautumispyyntöön itse
(`page.route`), eikä pyyntö koskaan mene backendille. Testi menisi siis läpi,
vaikka backend olisi rikki. Uusi testi käyttää oikeaa backendiä, joten se
huomaa myös virheet frontendin ja backendin välillä.

### Ajaminen

Backend käynnistetään ensin:

    cd backend
    .\mvnw spring-boot:run

Sen jälkeen testi ajetaan toisessa terminaalissa:

    cd frontend
    npx playwright test tests/system-login.spec.ts --project=chromium

Playwright käynnistää frontendin automaattisesti.

## Mitä opin

✍️ Kirjoita omin sanoin, esimerkiksi:
- mikä ero on mock-testillä ja järjestelmätestillä ja milloin kumpaakin kannattaa käyttää
- mitä kirjautumisessa tapahtuu frontendin ja backendin välillä
- mikä oli testin tekemisessä vaikeaa tai yllättävää

## Mitä tekisin toisin

✍️ Esimerkiksi: backendin käynnistyksen voisi automatisoida, jotta playwright testi
käynnistyisi yhdellä komennolla.

## Jatkokehitysideat

- Testi väärälle salasanalle (virheilmoitus näkyy, käyttäjä ei pääse sisään)
- Testi varauksen tekemiselle alusta loppuun
- Järjestelmätestin ajaminen automaattisesti GitHub Actionsissa

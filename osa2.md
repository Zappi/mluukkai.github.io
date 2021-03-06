---
layout: page
title: osa 2
permalink: /osa2/
---

## osan 2 oppimistavoitteet

- Web-sovellusten toiminnan perusteet
  - lisää CSS:ää
  - selain suoritusympäristönä
  - selaimen ja palvelimen välisen kummunikoinnin perusteet
- React
  - taulukossa olevan datan renderöinti
  - komponenttien määrittely moduuleissa
  - kontrolloidut lomakkeet
  - taulukossa olevan renderöitävän datan filtteröinti
  - komponentin 'lifecycle'-metodit
  - riippuvuuksien lisääminen
  - kommunikointi palvelimen kanssa
  - tyylien lisäämisen perusteita
- Javascript
  - template string
  - olioiden käsittelyä: property shorthand notation, object assign
  - ES6 moduulien perusteita
  - promiset
  - taulukoiden käsittelyä: map, filter, find

## kokoelmien renderöiminen

Tehdään nyt Reactilla [ensimmäisen osan](/osa1) alussa käytettyä esimerkkisovelluksen [Single page app -versiota](https://fullstack-exampleapp.herokuapp.com/spa) vastaavan sovelluksen 'frontend' eli selainpuolen koodi.

Aloitetaan seuraavasta:

```react
import React from 'react'
import ReactDOM from 'react-dom'

const notes = [
  {
    id: 1,
    content: 'HTML on helppoa',
    date: '2017-12-10T17:30:31.098Z',
    important: true
  },
  {
    id: 2,
    content: 'Selain pystyy suorittamaan vain javascriptiä',
    date: '2017-12-10T18:39:34.091Z',
    important: false
  },
  {
    id: 3,
    content: 'HTTP-protokollan tärkeimmät metodit ovat GET ja POST',
    date: '2017-12-10T19:20:14.298Z',
    important: true
  }
]

const App = (props) => {
  const { notes } = props;

  return (
    <div>
      <h1>Muistiinpanot</h1>
      <ul>
        <li>{note[0].content}</li>
        <li>{note[1].content}</li>
        <li>{note[2].content}</li>
      </ul>
    </div>
  )
}

ReactDOM.render(
  <App notes={notes} />,
  document.getElementById('root')
)
```

Jokaiseen muistiinpanoon on merkitty tekstuaalisen sisällön ja aikaleiman lisäksi myös _boolean_-arvo, joka kertoo onko muistiinpano luokiteltu tärkeäksi, sekä yksikäsitteinen tunniste _id_.

Koodin toiminta perustuu siihen, että taulukossa on tasan kolme muistiinpanoa, yksittäiset muistiinpanot renderöidään 'kovakoodatusti' viittaamalla suoraan taulukossa oleviin olioihin:

```html
<li>{note[1].content}</li>
```

Tämä ei tietenkään ole järkevää. Ratkaisu voidaan yleistää generoimalla taulukon perusteella joukko React-elementtejä käyttäen [map](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/map)-funktiota:

```js
notes.map(note => <li>{note.content}</li>)
```

nyt tuloksena on taulukko, jonka sisältö on joukko _li_-elementtejä

```js
[
  '<li>HTML on helppoa</li>',
  '<li>Selain pystyy suorittamaan vain javascriptiä</li>',
  '<li>HTTP-protokollan tärkeimmät metodit ovat GET ja POST</li>'
]
```

jotka voidaan sijoittaa _ul_-tagien sisälle:

```react
const App = (props) => {
  const { notes } = props;

  return (
    <div>
      <h1>Muistiinpanot</h1>
      <ul>
        {notes.map(note => <li>{note.content}</li>)}
      </ul>
    </div>
  )
}
```

Koska li-tagit generoiva koodi on javascriptia, tulee se sijoittaa JSX-templatessa aaltosulkujen sisälle kaiken muun javascript-koodin tapaan.

Usein vastaavissa tilanteissa dynaamisesti generoitava sisältö eristetään omaan metodiin, jota JSX-template kutsuu:

```react
const App = (props) => {
  const { notes } = props;
  const rivit = () => notes.map(note => <li>{note.content}</li>)

  return (
    <div>
      <h1>Muistiinpanot</h1>
      <ul>
        {rivit()}
      </ul>
    </div>
  )
}
```

Vaikka sovellus näyttää toimivan, tulee konsoliin ikävä varoitus

![]({{ "/assets/2/1.png" | absolute_url }})

Kuten virheilmoituksen linkittämä [sivu](https://reactjs.org/docs/lists-and-keys.html#keys) kertoo, tulee taulukossa olevilla, eli käytännössä _map_-metodilla muodostetuilla elementeillä olla uniikki avain, eli kenttä nimeltään _key_.

Lisätään avaimet:

```react
const App = (props) => {
  const { notes } = props;
  const rivit = () => notes.map(note => <li key={note.id}>{note.content}</li>)

  return (
    <div>
      <h1>Muistiinpanot</h1>
      <ul>
        {rivit()}
      </ul>
    </div>
  )
}
```

Virheilmoitus katoaa.

React käyttää taulukossa olevien elementtien key-kenttiä päätellessään miten sen tulee päivittää komponentin generoimaa näkymää silloin kun komponentti uudelleenrenderöidään. Lisää aiheesta [täällä](https://reactjs.org/docs/reconciliation.html#recursing-on-children).

### antipattern: taulukon indeksit avaimina

Olisimme saaneet konsolissa olevan varoituksen katoamaan myös käyttämällä avaimina taulukon indeksejä. Indeksit selviävät käyttämällä map-metodissa myös toista parametria:

```js
notes.map((note, i) => ...)
```

näin kutsuttaessa _i_ saa arvokseen sen paikan indeksin taulukossa, missä _note_ sijaitsee.

Eli virheilmoitukset poistuva tapa määritellä rivien generointi on

```js
const rivit = () => notes.map((note, i) => <li key={i}>{note.content}</li>)
```

Tämä **ei kuitenkaan ole suositeltavaa** ja voi näennäisestä toimimisestaan aiheuttaa joissakin tilanteissa pahoja ongelmia. Lue lisää esim. [täältä](https://medium.com/@robinpokorny/index-as-a-key-is-an-anti-pattern-e0349aece318).

### refaktorointia - moduulit

Siistitään koodia hiukan. Koska olemme kiinnostuneita ainoastaan propsien kentästä _notes_, otetaan se vastaan suoraan [destrukturointia](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment) hyödyntäen:

```react
const App = ({ notes }) => {
  return (
    <div>
      <h1>Muistiinpanot</h1>
      <ul>
        {rivit()}
      </ul>
    </div>
  )
}
```

Erotetaan yksittäisen muistiinpanon esittäminen oman komponenttinsa _Note_ vastuulle:

```react
const Note = ({ note }) => {
  return (
    <li>{note.content}</li>
  )
}

const App = ({ notes }) => {
  return (
    <div>
      <h1>Muistiinpanot</h1>
      <ul>
        {notes.map(note=><Note key={note.id} note={note}/>)}
      </ul>
    </div>
  )
}
```

Huomaa, että _key_-attribuutti täytyy nyt määritellä _Note_-komponenteille, eikä _li_-tageille kuten ennen muutosta.

Koko React-sovellus on mahdollista määritellä samassa tiedostossa, mutta se ei luonnollisesti ole järkevää. Usein käytäntönä on määritellä yksittäiset komponentit omassa tiedostossaan _ES6-moduuleina_.

Koodissamme on käytetty koko ajan moduuleja. Tiedoston ensimmäiset rivit

```js
import React from 'react'
import ReactDOM from 'react-dom'
```

[importtaavat](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/import) eli ottavat käyttöönsä kaksi moduulia. Moduuli _react_ sijoitetaan muuttujan _React_ ja _react-dom_ muuttujaan _ReactDOM_.


Siirretään nyt komponentti _Note_ omaan moduuliinsa.

Pienissä sovelluksissa komponentit sijoitetaan yleensä _src_-hakemiston alle sijoitettavaan hakemistoon _components_. Konventiona on nimetä tiedosto komponentin mukaan, eli tehdään hakemisto _components_ ja sinne tiedosto _Note.js_ jonka sisältö on seuraava:

```react
import React from 'react'

const Note = ({ note }) => {
  return (
    <li>{note.content}</li>
  )
}

export default Note
```

Koska kyseessä on React-komponentti, tulee React importata komponentissa.

Moduulin viimeisenä rivinä [eksportataan](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/export) määritelty komponentti, eli muuttuja _Note_.

Nyt komponenttia käyttävä tiedosto _index.js_ voi [importata](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/import) moduulin:

```react
import React from 'react'
import ReactDOM from 'react-dom'
import Note from './components/Note'
```

Moduulin eksporttaama komponentti on nyt käytettävissä muuttujassa _Note_ täysin samalla tavalla kuin aiemmin.

Koska myös _App_ on komponentti, eristetään sekin omaan moduuliinsa. Koska kyseessä on sovelluksen juurikomponentti, sijoitetaan se suoraan hakemistoon _src_, sisältö on seuraava:

```react
import React from 'react';
import Note from './components/Note'

const App = ({ notes }) => {
  return (
    <div>
      <h1>Muistiinpanot</h1>
      <ul>
        {notes.map(note => <Note key={note.id} note={note} />)}
      </ul>
    </div>
  )
}

export default App
```

Tiedoston _index.js_ sisällöksi jää:

```react
import React from 'react'
import ReactDOM from 'react-dom'
import App from './App'

const notes = [
  ...
]

ReactDOM.render(
  <App notes={notes} />,
  document.getElementById('root')
)
```
Moduuleilla on paljon muutakin käyttöä kuin mahdollistaa komponenttien määritteleminen omissa tiedostoissaan, palaamme moduuleihin tarkemmin myöhemmin kurssilla.

### Tehtäviä kokoelmien renderöinnistä

Tee nyt [tehtävät 21 - 25](../tehtavat#kokoelmien-renderöinti)

## Lomakkeet

Jatketaan sovelluksen laajentamista siten, että se mahdollistaa uusien muistiinpanojen lisäämisen.

Jotta saisimme sivun päivittymään uusien muistiinpanojen lisäyksen yhteydessä, on parasta sijoittaa muistiinpanot komponentin _App_ tilaan. Funktionaalisilla komponenteilla ei ole tilaa, joten muutetaan _App_ luokkaan perustuvaksi komponentiksi:

```react
class App extends React.Component {
  constructor(props) {
    super(props)
    this.state = {
      notes: props.notes
    }
  }

  render() {
    return (
      <div>
        <h1>Muistiinpanot</h1>
        <ul>
          {this.state.notes.map(note => <Note key={note.id} note={note} />)}
        </ul>
      </div>
    )
  }
}
```

Konstruktori asettaa nyt propseina saatavan _notes_-taulukon tilaan avaimen _notes_ arvoksi:

```js
  constructor(props) {
    super(props)
    this.state = {
      notes: props.notes
    }
  }
```

tila siis näyttää komponentin alustuksen jälkeen seuraavalta:

```js
this.state = {
  notes: [
    {
      id: 1,
      content: 'HTML on helppoa',
      date: '2017-12-10T17:30:31.098Z',
      important: true
    },
    //...
  ]
}
```

Lisätään sitten lomake uusen muistiinpanon lisäämistä varten:

```react
class App extends React.Component {
  constructor(props) {
    super(props)
    this.state = {
      notes: props.notes
    }
  }

  addNote = (e) => {
    e.preventDefault()
    console.log('nappia painettu')
  }

  render() {
    return (
      <div>
        <h1>Muistiinpanot</h1>
        <ul>
          {this.state.notes.map(note => <Note key={note.id} note={note} />)}
        </ul>
        <form onSubmit={this.addNote}>
          <input/>
          <button type="submit">tallenna</button>
        </form>
      </div>
    )
  }
}
```

Lomakkeelle on lisätty myös tapahtumankäsittelijäksi metodi _addNote_ reagoimaan sen "lähettämiseen", eli napin painamiseen.

Tapahtumankäsittelijä on [osasta 1](/osa1#tapahtumankäsittely) tuttuun tapaan määritelty seuraavasti:

```js
addNote = (e) => {
  e.preventDefault()
  console.log('nappia painettu')
  console.log(e.target)
}
```

Parametrin _e_ arvona on metodin kutsun aiheuttama [tapahtuma](https://reactjs.org/docs/handling-events.html).

Tapahtumankäsittelijä kutsuu heti tapahtuman metodia <code>e.preventDefault()</code> jolla se estää lomakkeen lähetyksen oletusarvoisen toiminnan, joka aiheuttaisi mm. sivun uudelleenlatautumisen.

Tapahtuman kohde, eli _e.target_ on tulostettu konsoliin

![]({{ "/assets/2/2.png" | absolute_url }})

Kohteena on siis komponentin määrittelemä lomake.

Miten pääsemme käsiksi lomakkeen _input_-komponenttiin syötettyyn dataan?

Tapoja on useampia, tutustumme ensin ns. [kontrolloituina komponentteina](https://reactjs.org/docs/forms.html#controlled-components) toteutettuihin lomakkeisiin.

Lisätään komponentin _App_ tilaan kenttä _new_note_ lomakkeen syötettä varten:

```js
class App extends React.Component {
  constructor(props) {
    super(props)
    this.state = {
      notes: props.notes,
      new_note: 'uusi muistiinpano...'
    }
  }
  // ...
}
```

Määritellään tilaan lisätty kenttä _input_-komponentin attribuutin _value_ arvoksi:

```html
<form onSubmit={this.addNote}>
  <input value={this.state.new_note} />
  <button type="submit">tallenna</button>
</form>
```

Tilaan määritelty "placeholder"-teksti  _uusi muistiinpano..._ ilmestyy syötekomponenttiin, tekstiä ei kuitenkaan voi muuttaa. Konsoliin tuleekin ikävä varoitus joka kertoo mistä on kyse

![]({{ "/assets/2/4.png" | absolute_url }})

Koska määrittelimme syötekomponentille _value_-attribuutiksi komponentin _App_ tilassa olevan kentän, alkaa _App_ [kontrolloimaan](https://reactjs.org/docs/forms.html#controlled-components) syötekomponentin toimintaa.

Jotta kontrolloidun syötekomponentin editoiminen olisi mahdollista, täytyy sille rekisteröidä _tapahtumankäsittelijä_, joka synkronoi syötekenttään tehdyt muutokset komponentin _App_ tilaan:

```react
class App extends React.Component {
  // ...

  handleNoteChange = (e) => {
    console.log(e.target.value)
    this.setState({ new_note: e.target.value })
  }

  render() {
    return (
      <div>
        <h1>Muistiinpanot</h1>
        <ul>
          {this.state.notes.map(note => <Note key={note.id} note={note} />)}
        </ul>
        <form onSubmit={this.addNote}>
          <input
            value={this.state.new_note}
            onChange={this.handleNoteChange}
          />
          <button type="submit" type="submit">tallenna</button>
        </form>
      </div>
    )
  }
}
```

Lomakkeen _input_-komponentille on nyt rekisteröity tapahtumankäsittelijä tilanteeseen _onChange_:

```html
<input
  value={this.state.new_note}
  onChange={this.handleNoteChange}
/>
```

Tapahtumankäsittelijää kutsutaan aina kun syötekomponentissa tapahtuu jotain. Tapahtumankäsittelijämetodi saa parametriksi tapahtumaolion _e_

```js
handleNoteChange = (e) => {
  console.log(e.target.value)
  this.setState({ new_note: e.target.value })
}
```

Tapahtumaolion kenttä _target_ vastaa nyt kontrolloitua _input_-kenttää ja _e.target.value_ viittaa inputin syötekentän arvoon. Voit seurata konsolista miten tapahtumankäsittelijää kutsutaan:

![]({{ "/assets/2/5.png" | absolute_url }})

Nyt komponentin _App_ tilan kenttä _new_note_ heijastaa koko ajan syötekentän arvoa, joten voimme viimeistellä uuden muistiinpanon lisäämisestä huolehtivan metodin _addNote_:

```js
addNote = (e) => {
  e.preventDefault()
  const noteObject = {
    content: this.state.new_note,
    date: new Date().new,
    important: Math.random() > 0.5
    id: this.state.notes.length + 1
  }

  const notes = this.state.notes.concat(noteObject)

  this.setState({
    notes: notes,
    new_note: ''
  })
}
```

Ensin luodaan uutta muistiinpanoa vastaava olio _noteObject_, jonka sisältökentän arvo saadaan komponentin tilasta _this.state.new_note_. Yksikäsitteinen tunnus eli _id_ generoidaan kaikkien muistiinpanojen lukumäärän perusteella. Koska muistiinpanoja ei poisteta, menetelmä toimii sovelluksessamme. Komennon <code>Math.random()</code> avulla muistiinpanosta tulee 50% todennäköisyydellä tärkeä.

Uusi muistiinpano lisätään vanhojen joukkoon oikeaoppisesti käyttämällä [osasta 1](/osa1#taulukon käsittelyä) tuttua taulukon metodia [concat](
https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/concat). Metodi ei muuta alkuperäistä taulukkoa _this.state.notes_ vaan luo uuden taulukon, joka sisältää myös lisättävän alkion. Tämä on tärkeää, sillä Reactin tilaa [ei saa muuttaa suoraan](https://reactjs.org/docs/state-and-lifecycle.html#using-state-correctly)!

Tila päivitetään uusilla muistiinpanoilla ja tyhjentämällä syötekomponentin arvoa kontrolloiva kenttä.

### Kehittyneempi tapa olioliteraalien kirjoittamiseen

Voimme muuttaa tilan päivittämän koodin

```js
this.setState({
  notes: notes,
  new_note: ''
})
```

muotoon

```js
this.setState({
  notes,
  new_note: ''
})
```

Tämä johtuu siitä, että ES6:n myötä (ks. kohta [property definitions](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Object_initializer)) javascriptiin on tullut uusi ominaisuus, joka mahdollistaa hieman tiiviimmän tavan määritellä olioita muuttujien avulla.

Tarkastellaan tilannetta, jossa meillä on muuttujissa arvoja

```js
const name: 'Leevi'
const age = 0
```

ja haluamme määritellä näiden perusteella olion, jolla on kentät _name_ ja _age_.

Vanhassa javascriptissä olio täytyi määritellä seuraavaan tapaan

```js
const person = {
  name: name,
  age: age
}
```

koska muuttujien ja luotavan olio kenttien nimi nyt on sama, riittää ES6:ssa kirjoittaa:

```js
const person = { name, age }
```

lopputulos molemmilla tavoilla luotuun olioon on täsmälleen sama.

## Näytettävien elementtien filtteröinti

Tehdään sovellukseen feature, joka mahdollistaa ainoastaan tärkeiden muistiinpanojen näyttämisen.

Lisätään komponentin _App_ tilaan tieto siitä näytetäänkö muistiinpanoista kaikki vai ainoastaan tärkeät:

```react
class App extends React.Component {
  constructor(props) {
    super(props)
    this.state = {
      notes: props.notes ,
      new_note: '',
      showAll: true
    }
  }
  // ...
}
```

Muutetaan metodia _render_ siten, että se tallettaa muuttujaan _notesToShow_ näytettävien muistiinpanojen listan riippuen siitä tuleeko näyttää kaikki vai vain tärkeät:

```react
render() {
  const notesToShow =
    this.state.showAll ?
      this.state.notes :
      this.state.notes.filter(note => note.important === true)

  return (
    <div>
      <h1>Muistiinpanot</h1>
      <ul>
        {notesToShow.map(note => <Note key={note.id} note={note} />)}
      </ul>
      <form onSubmit={this.addNote}>
        <input
          value={this.state.new_note}
          onChange={this.handleNoteChange}
        />
        <button type="submit">tallenna</button>
      </form>
    </div>
  )
}
```

Muuttujan _notesToShow_ määrittely on melko kompakti

```js
const notesToShow =
  this.state.showAll ?
    this.state.notes :
    this.state.notes.filter(note => note.important === true)
```

Käytössä on monissa muissakin kielissä oleva [ehdollinen](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Conditional_Operator) operaatio.

Operaatio toimi seuraavasti. Jos meillä on esim:

```js
const tulos = ehto ? val1 : val2
```

muuttujan _tulos_ arvoksi asetetaan _val1_:n arvo jos _ehto_ on tosi. Jos _ehto_ ei ole tosi, muuttujan _tulos_ arvoksi tulee _val2_:n arvo.

Jos ehto _this.state.showAll_ on epätosi, muuttuja _notesToShow_ saa arvokseen vaan ne muistiinpanot, joiden _important_-kentän arvo on tosi. Filtteröinti tapahtuu taulukon metodilla [filter](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/filter):

```js
this.state.notes.filter(note => note.important === true)
```

vertailu-operaatio on oikeastaan turha koska _note.important_ on arvoltaan joko _true_ tai _false_, eli riittää kirjoittaa

```js
this.state.notes.filter(note => note.important)
```

Tässä käytettiin kuitenkin ensin vertailuoperaattoria, mm. korostamaan erästä tärkeää seikkaa: Javascriptissa <code>arvo1 == arvo2</code> ei toimi kaikissa tilanteissa loogisesti ja onkin varmempi käyttää aina vertailuissa muotoa <code>arvo1 === arvo2</code>. Enemmän aiheesta [täällä](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Equality_comparisons_and_sameness).

Filtteröinnin toimivuutta voi jo nyt kokeilla vaihtelemalla sitä, miten tilan kentän _showAll_ alkuarvo määritelään konstruktorissa.

Lisätään sitten toiminnallisuus, mikä mahdollistaa _showAll_:in tilan muuttamisen sovelluksesta.

Oleelliset muutokset ovat seuraavassa:

```react
class App extends React.Component {
  // ...

  toggleVisible = () => {
    this.setState({showAll: !this.state.showAll})
  }

  render() {
    const notesToShow =
      this.state.showAll ?
        this.state.notes :
        this.state.notes.filter(note => note.important === true)

    const label = this.state.showAll ? 'vain tärkeät' : 'kaikki'

    return (
      <div>
        <h1>Muistiinpanot</h1>

        <div>
          <button onClick={this.toggleVisible}>
            näytä {label}
          </button>
        </div>

        <ul>
          {notesToShow.map(note => <Note key={note.id} note={note} />)}
        </ul>
        <form onSubmit={this.addNote}>
          <input
            value={this.state.new_note}
            onChange={this.handleNoteChange}
          />
          <button type="submit">tallenna</button>
        </form>
      </div>
    )
  }
}
```

Näkyviä muistiinpanoja (kaikki vai ainoastaan tärkeät) siis kontrolloidaan napin avulla. Napin tapahtumankäsittelijä on yksinkertainen, se muuttaa _this.state.showAll_:n arvon truesta falseksi ja päinvastoin:

```js
toggleVisible = () => {
  this.setState({showAll: !this.state.showAll})
}
```

Napin teksti määritellään muuttujaan, jonka arvo määräytyy tilan perusteella:

```js
const label = this.state.showAll ? 'vain tärkeät' : 'kaikki'
```

### Tehtäviä lomakkeista

Tee nyt tehtävät [26-30](../tehtavat#lomakkeet)

## Datan haku palvelimelta

Olemme nyt viipyneet tovin keskittyen pelkkään "frontendiin", eli selainpuolen toiminnallisuuteen. Rupeamme itse toteuttamaan "backendin", eli palvelinpuolen toiminnallisuutta vasta kurssin kolmannessa osassa, mutta otamme nyt jo askeleen sinne suuntaan tutustumalla siihen miten selaimessa suoritettava koodi kommunikoi backendin kanssa.

Käytetään nyt palvelimena sovelluskehitykseen tarkoitettua [JSON Serveriä](https://github.com/typicode/json-server).

[Asenna](https://github.com/typicode/json-server#install) JSON server.

Tee projektin juurihakemistoon tiedosto _db.json_, jolla on seuraava sisältö:

```js
{
  "notes": [
    {
      "id": 1,
      "content": "HTML on helppoa",
      "date": "2017-12-10T17:30:31.098Z",
      "important": true
    },
    {
      "id": 2,
      "content": "Selain pystyy suorittamaan vain javascriptiä",
      "date": "2017-12-10T18:39:34.091Z",
      "important": false
    },
    {
      "id": 3,
      "content": "HTTP-protokollan tärkeimmät metodit ovat GET ja POST",
      "date": "2017-12-10T19:20:14.298Z",
      "important": true
    }
  ]
}
```

Käynnistä _json-server_ porttiin 3001:

```bash
json-server --port=3001 --watch db.json
```

Oletusarvoisesti _json-server_ käynnistyy porttiin 3000, mutta create-react-app:illa luodut projektit varaavat portin 3000, joten joudumme nyt määrittelemään json-server:ille vaihtoehtoisen portin.

Mennään selaimella osoitteeseen <http://localhost:3001/notes>. Kuten huomaamme, _json-server_ tarjoaa osoitteessa tiedostoon tallentamamme muistiinpanot:

![]({{ "/assets/2/6.png" | absolute_url }})

Ideana jatkossa onkin se, että muistiinpanot talletetaan palvelimelle, eli tässä vaiheessa _json-server_:ille. React-koodi lataa muistiinpanot palvelimelta ja renderöi ne ruudulle. Kun sovellukseen lisätään uusi muistiinpano, React-koodi lähettää sen myös palvelimelle, jotta uudet muistiinpanot jäävät pysyvästi "muistiin".

json-server tallettaa kaiken datan palvelimella sijaitsevaan tiedostoon _db.json_. Todellisuudessa data tullaan tallentamaan johonkin tietokantaan. json-server on kuitenkin käyttökelpoinen apuväline, joka mahdollistaa palvelinpuolen toiminnallisuuden käyttämisen kehitysvaiheessa ilman tarvetta itse ohjelmoida mitään.

Tutustumme palvelinpuolen toteuttamisen periaatteisiin tarkemmin kurssin [osassa 3](/osa3).

### Selain suoritusympäristönä

Ensimmäisenä tehtävänämme on siis hakea React-sovellukseen jo olemassaolevat mustiinpano osoitteesta <http://localhost:3001/notes>.


Osan 1 [esimerkkiprojektissa](osa1/#elaimessa-suoritettava-sovelluslogiikka) nähtiin jo eräs tapa hakea javascript-koodista palvelimella olevaa dataa. Esimerkin koodissa data haettiin [XMLHttpRequest](https://developer.mozilla.org/en-US/docs/Web/API/XMLHttpRequest) eli XHR-olion avulla muodostetulla HTTP-pyynnöllä. Kyseessä on 1999 lanseerattu tekniikka, jota kaikki web-selaimet ovat jo pitkään tukeneet.

Nykyään XHR:ää ei kuitenkaan kannata juurikaan käyttää ja selaimet tukevatkin jo laajasti [fetch](https://developer.mozilla.org/en-US/docs/Web/API/WindowOrWorkerGlobalScope/fetch)-metodia, joka perustuu XHR:n käyttämän tapahtumapohjaisen mallin sijaan ns. [promiseja](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise).

Muistutuksena viime viikosta (oikeastaan tätä tapaa pitää lähinnä _muistaa olla käyttämättä_ ilman painavaa syytä), XHR:llä haettiin dataa seuraavasti

```bash
const xhttp = new XMLHttpRequest()

xhttp.onreadystatechange = function () {
  if (this.readyState == 4 && this.status == 200) {
    const data = JSON.parse(this.responseText)
    // käsittele muuttujaan data sijoitettu kyselyn tulos
  }
}

xhttp.open('GET', '/data.json', true)
xhttp.send()
```

Heti alussa HTTP-pyyntöä vastaavalle _xhttp_-oliolle rekisteröidään tapahtumankäsittelijä, jota javascript runtime kutsuu kun _xhttp_-olion tila muuttuu. Jos tilanmuutos tarkoittaa että pyynnön vastaus on saapunut, käsitellään data halutulla tavalla.

Huomionarvoista on se, että tapahtumankäsittelijän koodi on määritelty jo ennen kun itse pyyntö lähetetään palvelimelle. Tapahtumankäsittelijäfunktio tullaan kuitenkin suorittamaan vasta jossain myöhäisemmässä vaiheessa. Koodin suoritus ei siis etene synkronisesti "ylhäältä alas", vaan _asynkronisesti_, javascript kutsuu sille rekisteröityä tapahtumankäsittelijäfunktiota jossain vaiheessa.

Esim. Java-ohjelmoinnista tuttu synkroninen tapa tehdä kyselyjä etenisi seuraavaan tapaan (huomaa että kyse ei ole oikeasti toimivasta Java-koodista):

```java
HTTPRequest request = new HTTPRequest()

List<Muistiinpano> muistiinpanot = request.get("https://fullstack-exampleapp.herokuapp.com/data.json");

muistiinpanot.forEach(m => {
  System.out.println(m.content);
})
```

Javassa koodi etenee nyt rivi riviltä ja koodi pysähtyy odottamaan HTTP-pyynnön, eli komennon _request.get(...)_ valmistumista. Komennon palauttama data, eli muistiinpanot talletetaan muuttujaan ja dataa aletaan käsittelemään halutulla tavalla.

Javascript-enginet eli suoritusympäristöt kuitenkin noudattavat [asynkronista mallia](https://developer.mozilla.org/en-US/docs/Web/JavaScript/EventLoop), eli periaatteena on se, että kaikki [IO-operaatiot](https://en.wikipedia.org/wiki/Input/output) (poislukien muutama poikkeus) suoritetaan ei-blokkaavana, eli operaatioiden tulosta ei jäädä odottamaan vaan koodin suoritusta jatketaan heti eteenpäin.

Siinä vaiheessa kun operaatio valmistuu tai tarkemmin sanoen jonain valmistumisen jälkeisenä ajanhetkenä, kutsuu Javascript-engine operaatiolle rekisteröityjä tapahtumankäsittelijöitä.

Nykyisellään javascript-moottorit ovat _yksisäikeisiä_ eli ne eivät voi suorittaa rinnakkaista koodia. Tämän takia on käytännössä pakko käyttää ei-blokkaavaa maillia IO-operaatioiden suorittamiseen, sillä muuten esim. selain 'jäätyisi' siksi aikaa kun esim. palvelimelta haetaan dataa.

Javasript-moottoreiden yksisäikeisyydellä on myös sellainen seuraus, että jos koodin suoritus kestää erittäin pitkään, selain jäätyy suorituksen ajaksi. Jos lisätään jonnekin kohtaa sovellustamme, esim. konstruktoriin seuraava koodi:

```
setTimeout(() => {
  console.log('loop..')
  let i = 0
  while(i < 50000000000) {
    i++
  }
  console.log('end')
}, 5000)
```

Kaikki toimii 5 sekunnin ajan normaalisti. Kun _setTimeout_:in parametrina määritelty funktio suoritetaan, menee selaimen sivu jumiin pitkän loopin suorituksen ajaksi. Ainakaan Chromessa selaimen tabia ei pysty edes sulkemaan luupin suorituksen aikana.

Eli jotta selain säilyy _responsiivisena_, eli että se reagoi koko ajan riittävän nopeasti käyttäjän haluamiin toimenpiteisiin, koodin logiikan tulee olla sellainen, että yksittäinen laskenta ei saa kestää liian kauaa.

Aiheesta löytyy paljon lisämateriaalia internetistä, eräs varsin havainnollinen esitys aiheesta Philip Robertsin esitelmä [What the heck is the event loop anyway?](https://www.youtube.com/watch?v=8aGhZQkoFbQ)

Nykyään selaimissa on mahdollisuus suorittaa myös rinnakkaista koodia ns. [web workerien](https://developer.mozilla.org/en-US/docs/Web/API/Web_Workers_API/Using_web_workers) avulla. Yksittäisen selainikkunan koodin ns. event loopista huolehtii kuitenkin edelleen [vain yksi säie](https://medium.com/techtrument/multithreading-javascript-46156179cf9a).

## npm

Palaamme jälleen asiaan, eli datan hakemiseen palvelimelta.

Voisimme käyttää datan palvelimelta hakemiseen aiemmin mainittua promiseihin perustuvaa funktiota [fetch](https://developer.mozilla.org/en-US/docs/Web/API/WindowOrWorkerGlobalScope/fetch). Fetch on hyvä työkalu, se on standardoitu ja kaikkien modernien selaimien (poislukien IE) tukema.

Käytetään selaimen ja palvelimen väliseen kommunikaatioon kuitenkin [axios](https://github.com/axios/axios)-kirjastoa, joka toimii samaan tapaan kuin fetch, mutta on hieman mukavampikäyttöinen. Hyvä syy axios:in käytölle on myös se, että pääsemme tutustumaan siihen miten ulkopuolisia kirjastoja eli _npm-paketteja_ liitetään React-projektiin.

Nykyään lähes kaikki Javascript-projektit määritellään node "pakkausmanagerin" eli [npm](https://docs.npmjs.com/getting-started/what-is-npm):n avulla. Myös create-react-app:in avulla generoidut projektit ovat npm-muotoisia projekteja. Varma tuntomerkki siitä on projektin juuressa oleva tiedosto _package.json_:

```js
{
  "name": "osa2",
  "version": "0.1.0",
  "private": true,
  "dependencies": {
    "react": "^16.2.0",
    "react-dom": "^16.2.0",
    "react-scripts": "1.0.17"
  },
  "scripts": {
    "start": "react-scripts start",
    "build": "react-scripts build",
    "test": "react-scripts test --env=jsdom",
    "eject": "react-scripts eject"
  }
}
```

Tässä vaiheessa meitä kiinnostaa osa _dependencies_, joka määrittelee mitä _riippuvuuksia_ eli ulkoisia kirjastoja projektilla on.

Haluamme nyt käyttöömme axioksen. Voisimme määritellä kirjaston suoraan tiedostoon _package.json_, mutta on parempi asentaa se komentoriviltä

```bash
npm install axios --save
```

Nyt axios on mukana riippuvuuksien joukossa:

```js
{
  "dependencies": {
    "axios": "^0.17.1",
    "react": "^16.2.0",
    "react-dom": "^16.2.0",
    "react-scripts": "1.0.17"
  },
  /*...*/
}
```

Sen lisäksi, että komento _npm install_ lisäsi axiosin riippuvuuksien joukkoon, se myös _latasi_ kirjaston koodin. Koodi löytyy muiden riippuvuuksien tapaan projektin juuren hakemistosta _node_modules_, mikä kuten huomata saattaa sisältääkin runsaasti kaikenlaista.

Tutustumme npm:n tarkemmin kurssin [kolmannessa osassa](/osa3).

### Axios ja promiset

Olemme nyt valmiina käyttämään axiosia. Jatkossa oletetaan että _json-server_ on käynnissä portissa 3001.

Kirjaston voi ottaa käyttöön samaan tapaan kuin esim. React otetaan käyttöön, eli sopivalla _import_-lauseella.

Lisätään seuraava tiedotoon _index.js_

```js
import axios from 'axios'

const promise = axios.get('http://localhost:3001/notes')
console.log(promise)

const promise2 = axios.get('http://localhost:3001/foobar')
console.log(promise2)
```

![]({{ "/assets/2/8.png" | absolute_url }})

Axiosin metodi _get_ palauttaa [promisen](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Using_promises).

Mozillan dokumentaatio sanoo promisesta seuraavaa:
> A Promise is an object representing the eventual completion or failure of an asynchronous operation. Since most people are consumers of already-created promises, this guide will explain consumption of returned promises before explaining how to create them.

Promise siis edustaa asynkronista operaatiota. Promise voi olla kolmessa eri tilassa:
* aluksi promise on _pending_, eli promisea vastaava asynkroninen operaatio ei ole vielä tapahtunut
* jos operaatio päätyy onnistuneesti, menee promise tilaan _fulfilled_, josta joskus käytetään nimitystä _resolved_
* kolmas mahdollinen tila on _rejected_, joka edustaa epäonnistunutta operaatiota

Esimerkkimme ensimmäinen promise on _fulfilled_, eli vastaa vastaa onnistunutta _axios.get('http://localhost:3001/notes')_ pyyntöä. Promiseista toinen taas on _rejected_, syy selviää konsolista, eli yritettiin tehdä HTTP GET -pyyntöä osoitteeseen, jota ei ole olemassa.

Jos ja kun haluamme tietoon promisea vastaavan operaation tuloksen, tulee promiselle rekisteröidä tapahtumankuuntelija. Tämä tapahtuu metodilla _then_:

```
const promise = axios.get('http://localhost:3001/notes')

promise.then(response => {
  console.log(response)
})
```

Konsoliin tulostuu seuraavaa

![]({{ "/assets/2/9.png" | absolute_url }})

Javascriptin suoritusympäristö kutsuu _then_-metodin avulla rekisteröityä takaisinkutsufunktiota antaen sille parametriksi olion _result_, joka sisältää kaiken oleellisen HTTP GET -pyynnön vastaukseen liittyvän, eli palautetun _datan_, _statuskoodin_ ja _headerit_.

Promise-olioa ei ole yleensä tarvetta tallettaa muuttujaan, ja onkin tapana ketjuttaa metodin _then_ kutsu suoraan axiosin metodin kutsun perään:


```js
axios.get('http://localhost:3001/notes').then(response => {
  const notes = response.data
  console.log(notes)
})
```

Takaisinkutsufunktio ottaa nyt vastauksen sisällä olevan datan muuttujaan ja tulostaa muistiinpanot konsoliin.

Palvelimen palauttama data on pelkkää tekstiä, käytännössä yksi iso merkkijono. Asian voi todeta, esim. tekemällä HTTP-pyyntö komentoriviltä [curl](https://curl.haxx.se):illa

![]({{ "/assets/2/10.png" | absolute_url }})


Axios-kirjasto osaa kuitenkin parsia datan Javascript-taulukoksi, sillä palvelin on kertonut headerin _content-type_ avulla että datan muoto on _application/json; charser=utf-8_ (ks ylempi kuva).

Voimme vihdoin siirtyä käyttämään sovelluksessamme palvelimelta haettavaa dataa.

Tehdään se aluksi "huonosti", eli lisätään sovellusta vastaavan komponentin _App_ renderöinti takaisinkutsufunktion sisälle:

```react
axios.get('http://localhost:3001/notes').then(response => {
  const notes = response.data
  ReactDOM.render(
    <App notes={notes} />,
    document.getElementById('root')
  )
})
```

Joissain tilanteissa tämäkin tapa voisi olla ok, mutta se on hieman ongelmallinen ja päätetäänkin siirtää datan hakeminen komponenttiin _App_.

Ei ole kuitenkaan ihan selvää, mihin kohtaan komponentin koodia komento _axios.get_ olisi hyvä sijoittaa.

### Komponenttien lifecycle-metodit

Reactin luokkien avulla määritellyillä komponenteilla voidaan määritellä joukko [lifecycle](https://reactjs.org/docs/state-and-lifecycle.html#adding-lifecycle-methods-to-a-class)-metodeita, eli metodeita, joita React kutsuu tietyssä komponentin "elinkaaren" vaiheessa.

Yleinen tapa datan palvelimelta tapahtuvaan lataamiseen onsuorittaa se metodissa [componentWillMount](https://reactjs.org/docs/react-component.html#componentwillmount). React kutsuu metodia sen jälkeen kun konstruktori on suoritettu ja _render_-metodia ollaan kutsumassa ensimmäistä kertaa.

Muutetaan sovellusta nyt seuraavasti.

Poistetaan datan hakeminen tiedostosta _index.js_:

```js
ReactDOM.render(
  <App />,
  document.getElementById('root')
)
```

Komponentille _App_ ei ole enää tarvetta välittää dataa propseina.

Komponentti _App_ muuttuu seuraavasti:

```react
import React from 'react'
import axios from 'axios'

class App extends React.Component {
  constructor(props) {
    super(props)
    this.state = {
      notes: [],
      new_note: '',
      showAll: true
    }
    console.log('constructor')
  }

  componentWillMount() {
    console.log('will mount')
    axios.get('http://localhost:3001/notes')
      .then(response => {
        console.log('promise fulfilled')
        this.setState({ notes: response.data })
      })
  }

  render() {
    console.log('render')
    // ...
  }
}
```

Eli konstruktorissa asetetaan tilan _notes_ kentäksi tyhjä taulukko. Lifecycle-metodi _componentWillMount_ hakee datan axiosin avulla ja rekisteröi takaisinkutsufunktion, joka promisen valmistumisen (_fulfillment_) yhteydessä päivittää komponentin tilan asettamalla palvelimen palauttamat muistiinpanot tilan kentän _notes_ arvoksi.

Koodiin on myös lisätty muutama aputulostus, jotka auttavat hahmottamaan miten suoritus etenee.

Konsoliin tulostuu

<pre>
constructor
will mount
render
promise fulfilled
render
</pre>

Ensin siis suoritetaan konstruktori, ja sen jälkeen metodi _componentWillMount_. Tämän jälkeen kutsutaan kuitenkin metodia _render_, miksi näin?

Metodissa _componentWillMount_ suoritetaan axiosin avulla HTTP GET -pyyntö ja samalla _rekisteröidään_ pyynnön palauttamalle promiselle tapahtumankäsittelijä:

```js
axios.get('http://localhost:3001/notes').then(response => {
  console.log('promise fulfilled')
  this.setState({ notes: response.data })
})
```

Tapahtumankäsittelijän koodia, eli then:in parametrina olevaa _funktiota_ ei siis suoriteta vielä tässä vaiheessa. Javascriptin runtime kutuu sitä jossain vaiheessa sen jälkeen kun palvelin on vastannut HTTP GET -pyyntöön. Tätä ennen kutsutaan metodia _render_ ja komponenentti _App_ piirtyy ruudulle aluksi siten, että yhtään muistiinpanoa ei näytetä.

Emme kuitenkaan ehdi huomaammaan asiaa, sillä palvelimen vastaus tulee pian, ja se taas saa aikaan tapahtumankäsittelijän suorituksen. Tapahtumankäsittelijä päivittää komponentin tilaa kutsumalla _setState_ ja tämä saa aikaan komponentin uudelleenrenderöinnin.

Mieti tarkasti äsken läpikäytyä tapahtumasarjaa, sen ymmärtäminen on erittäin tärkeää!

Huomaa, että olisimme voineet kirjoittaa koodin myös seuraavasti:

```js
const eventHandler = (response) => {
  console.log('promise fulfilled')
  this.setState({ notes: response.data })
}

const promise = axios.get('http://localhost:3001/notes')

promise.then(eventHandler)
```

Muuttujaan _eventHandler_ on sijoitettu viite funktioon. Axiosin metodin get palauttama promise on talletettu muuttujaan _promise_. Takaisinkutsun rekisteröinti tapahtuu antamalla promisen then-metodin parametrina muuttuja _eventHandler_, joka viittaa käsittelijäfunktioon.

React-komponenteilla on myös joukko muita [lifecycle-metodeja](https://reactjs.org/docs/react-component.html), palaamme niihin myöhemmin.

### Tehtäviä datan hakemisesta palvelimelta

Tee nyt tehtävä [31-33](../tehtavat#datan-hakeminen-palvelimelta)

## REST API:n käyttö

Kun sovelluksella luodaan uusia muistiinpanoja, täytyy ne tallentaa palvelimelle.

json-server mainitsee olevansa ns. REST tai RESTful API

> Get a full fake REST API with zero coding in less than 30 seconds (seriously)

Ihan alkuperäisen [määritelmän](https://en.wikipedia.org/wiki/Representational_state_transfer) mukainen RESTful API json-server ei ole, mutta ei ole kovin moni muukaan itseään REST:iksi kutsuva rajapinta.

Tutustumme REST:iin tarkemmin kurssin [seuraavassa osassa](/osa3), mutta jo nyt on tärkeä ymmärtää minkälaista [konventiota](https://en.wikipedia.org/wiki/Representational_state_transfer#Applied_to_Web_services) json-server ja yleisemminkin REST API:t käyttävät [reittien](https://github.com/typicode/json-server#routes), eli URL:ien ja käytettävien HTTP-pyyntöjen tyyppien suhteen.

REST:issä yksittäisi asioita esim. meidän tapauksessamme muistiinpanoja kutsutaan _resursseiksi_. Jokaisella resurssilla on yksilöivä osoite eli URL. json-serverin noudattaman yleisen konvention mukaan yksittäisen muistiinpanoa kuvaavan resurssin URL on muotoa _notes/3_, missä 3 on resurssin tunniste. Osoite _notes_ taas vastaa kaikkien yksittäisten muistiinpanojen kokoelmaa.

Resursseja haetaan palvelimelta HTTP GET -pyynnöillä. Esim. HTTP GET osoitteeseen _notes/3_ palauttaisi muistiinpanon, jonka id-kentän arvo on 3. Kun taan HTTP GET -pyyntö osoitteeseen _notes_ palauttaa kaikki muistiinpanot.

Uuden muistiinpanoa vastaavan resurssin luominen tapahtuu json-serverin RESTful-konventiossa tekemällä HTTP POST -pyyntö, joka kohdistuu myös samaan osoiteeseen _notes_. Pyynnön mukana sen runkona eli _bodynä_ lähetetään luotavan muistiinpanon tiedot.

json-server vaatii, että tiedot lähetetään JSON-muodossa, eli käytännössä sopivasti muotoiltuna merkkijonona ja asettamalla headerille _Content-Type_ arvo _application/json_.

## Datan lähetys palvelimelle

Muutetaan nyt uuden muistiinpanon lisäämisestä huolehtivaa tapahtumankäsittelijää seuraavasti:

```js
addNote = (e) => {
  e.preventDefault()
  const noteObject = {
    content: this.state.new_note,
    date: new Date().new,
    important: Math.random() > 0.5,
  }

  axios.post('http://localhost:3001/notes', noteObject)
    .then(response => {
      console.log(response)
    })
}
```

eli luodaan muistiinpanoa vastaava olio, ei kuitenkaan lisätä sille kenttää _id_, parempi jättää id:n generointi palvelimen vastuulle!

Lähetetään sitten olio palvelimelle käyttämällä axiosin metodia _post_. Rekisteröidään tapahtumankäsittelijä, joka tulostaa konsoliin palvelimen vastauksen.

Kun nyt kokeillaan luoda uusi muistiinpano, konsoliin tulostus näyttää seuraavalta:

![]({{ "/assets/2/11.png" | absolute_url }})

Uusi muistiinpano on siis _response_-olion kentän _data_ arvona. Palvelin on lisännyt muistiinpanolle tunnisteen, eli _id_-kentän.

Joskus on hyödyllistä tarkastella HTTP-pyyntöjä [osan 1 alussa](osa1/#HTTP-GET) paljon käytetyn konsolin _Network_-välilehden kautta:

![]({{ "/assets/2/12.png" | absolute_url }})

Voimme, esim. tarkastaa onko POST-pyynnön mukana menevä data juuri se mitä oletimme, onko headerit asetettu oikein ym.

Koska POST-pyynnössä lähettämämme data oli javascrip-olio, osasi axios automaattisesti asettaa pyynnön _content-type_ headerille oikean arvon eli _application/json_.

Uusi muistiinpano ei vielä renderöidy ruudulle, sillä emme aseta komponentille _App_ uutta tilaa muistiinpanon luomisen yhteydessä. Viimeistellään sovellus vielä tältä osin:

```js
addNote = (e) => {
  e.preventDefault()
  const noteObject = {
    content: this.state.new_note,
    date: new Date(),
    important: Math.random() > 0.5,
  }

  axios.post('http://localhost:3001/notes', noteObject)
    .then(response => {
      this.setState({
        notes: this.state.notes.concat(response.data),
        new_note: ''
      })
    })
}
```

Palvelimen palauttama uusi muistiinpano siis lisätään tilassa olevien muiden muistiinpanojen joukkoon (kannattaa [muistaa tärkeä detalji](osa1/#taulukon-käsittelyä) siitä, että metodi _concat_ ei muuta komponentin alkuperäistä tilaa, vaan luo uuden uuden taulukon) ja tyhjennetään lomakkeen teksti.

Kun palvelimella oleva data alkaa vaikuttaa web-sovelluksen toimintalogiikkaan, tulee sovelluskehitykseen heti iso joukko uusia haasteita, joita tuo mukanaan mm. kommunikoinnin asynkroonisuus. Debuggaamiseenin tarvitaan uusia strategiota, debug-printtaukset ym muuttuvat vain tärkeämmäksi, myös javascriptin runtimen periaatteita ja React-komponenttien elinkaarta on pakko tuntea riittävällä tasolla, arvaileminen ei riitä.

Palvelimen tilaa kannattaa tarkastella myös suoraan, esim. selaimella:

![]({{ "/assets/2/13.png" | absolute_url }})

näin on mahdollista varmistua, mm. siirtyykö kaikki oletettu data palvelimelle.

Kurssin seuraavassa osassa alamme toteuttaa itse myös palvelimella olevan sovelluslogiikan, tutustumme silloin tarkemmin palvelimen debuggausta auttaviin työkaluihin, mm. [postmaniin](https://chrome.google.com/webstore/detail/postman/fhbjgbiflinjbdggehcddcbncdddomop). Tässä vaiheessa json-server-palvelimen tilan tarkkailuun riittänee selain.

## Muistiinpanon tärkeyden muutos

Lisätään muistiinpanojen yhteyteen painike, millä niiden tärkeyttä voi muuttaa.

Muistiinpanon määrittelevän komponentin muutos on seuraava:

```js
const Note = ({note, toggleImportance}) => {
  const label = note.important ? 'make not important' : 'make important'
  return (
    <li>{note.content} <button onClick={toggleImportance}>{label}</button></li>
  )
}
```

Komponentissa on nappi, jolle on rekisteröity klikkaustapahtuman käsittelijäksi propsien avulla välitetty funktio _toggleImportance_.

Tapahtumankäsittelijän alustava versio on määritelty komponentissa _App_ seuraavasti:

```js
toggleImportanceOf = (id) => {
  return () => {
    console.log('importance of '+id+' needs to be toggled')
  }
}
```

Kyseessä on jälleen funktio, joka palauttaa funktion. Palataan sen sisälttöön kohta.

Komponentin _App_ metodissa _render_ välitetään jokaiselle muistiinpanolle tapahtumankäsittelijäfunktio:

```html
<ul>
  {notesToShow.map(note =>
    <Note
      key={note.id}
      note={note}
      toggleImportance={this.toggleImportanceOf(note.id)}
    />
  )}
</ul>
```

Jokaisen muistiinpanon tapahtumankäsittelijä on nyt _yksilöllinen_, sillä se sisältää muistiinpanon _id:n_. Esim. jos _note.id_ on 3 tulee tapahtumankäsittelijäksi _this.toggleImportance(note.id)_ eli käytännössä:

```js
() => {
  console.log('importance of 3 needs to be toggled')
}
```

Eli komponentin _App_ metodi _toggleImportanceOf_ ei itsessään ole tapahtumankäsittelijä, vaan _tehdas_, jonka avulla kullekin muistiinpanolle luodaan oma tapahtumankäsittelijä.

Pieni huomio tähän väliin. Tapahtumankäsittelijän koodin tulostuksessa muodostetaan tulostettava merkkijono Javan tyyliin plussaamalla stringejä:

```js
console.log('importance of '+id+' needs to be toggled')
```

ES6:n [template string](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Template_literals) -ominaisuuden ansiosta Javascriptissa vastaavat merkkijonot voidaan kirjottaa hieman mukavammin:

```js
console.log(`importance of ${id} needs to be toggled`)
```

Merkkijonon sisälle voi nyt määritellä "dollari-aaltosulku"-syntaksilla kohtia, minkä sisälle evaluoidaan javascript-lausekkeita, esim. muuttujan arvo. Huomaa, että template stringien hipsutyyppi poikkeaa javascriptin normaaleista merkkijonojen käyttämistä hipsuista.

Yksittäistä json-serverillä olevaa muistiinpanoa voi muuttaa kahdella tavalla, joko _korvaamalla_ sen tekemällä HTTP PUT -pyyntö muistiinpanon yksilöivään osoitteeseen tai muu muuttamalla ainoastaan joidenkin muistiinpanon kenttien arvoja HTTP PATCH -pyynnöllä.

Korvaamme nyt muistiinpanon kokonaan, sillä samalla tulee esille muutama tärkeä React:iin ja Javascriptiin liittyvä seikka.

Metodi on seuraavassa:

```js
toggleImportanceOf = (id) => {
  return () => {
    const url = `http://localhost:3001/notes/${id}`
    const note = this.state.notes.find(n => n.id === id)
    const changedNote = { ...note, important: !note.important }

    axios.put(url, changedNote)
      .then(response => {
        const nonchangedNotes = this.state.notes.filter(n => n.id !== id)
        this.setState({
          notes: nonchangedNotes.concat(response.data)
        })
    })
  }
}
```

Melkein joka riville sisältyy tärkeitä yksityiskohtia. Ensimmäinen rivi määrittelee jokaiselle muistiinpanolle id-kenttään perustuvan yksilöivän url:in.

Taulukon metodilla [find](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/find) etsitään muutettava muistiinpano ja talletetaan viite siihen muuttujaan _note_.

Sen jälkeen luodaan _uusi olio_, jonka sisältö on sama kuin vanhan olion sisältö poislukien kenttä important. Luominen näyttää hieman erikoiselta:

```js
const changedNote = { ...note, important: !note.important }
```

Kyseessä on vielä standardoimattoman [object spread](https://github.com/tc39/proposal-object-rest-spread) -operaation soveltaminen.

Käytännössä <code>{...note}</code> luo olion, jolla on kenttinään kopiot olion _note_ kenttien arvoista. Kun aaltosulkeisiin lisätään asioita, esim. <code>{ ...note, important: true }</code>, tulee uuden olion kenttä _important_ saamaan arvon _true_. Eli esimerkissämme _important_ saa uudessa oliossa vanhan arvonsa käänteisarvon.

Uusi olio olisi voitu luodan myös vanhemmalla komennolla [Object.assign](https://developer.mozilla.org/nl/docs/Web/JavaScript/Reference/Global_Objects/Object/assign)

```js
const changedNote = Object.assign({}, note, {important: !note.important} }
```

Object spread -syntaksi on kuitenkin yleisesti käytössä Reactissa, joten mekin käytämme sitä.

Pari huomioita. Miksi teimme muutettavasta oliosta kopion vaikka myös seuraava koodi näyttää toimivan:

```js
const note = this.state.notes.find(n => n.id === id)
note.important = !note.important

axios.put(url, changedNote).then(response => {
```

Näin ei ole suositetavaa tehdä, sillä muuttuja _note_ on viite komponentin tilassa, eli _this.state.notes_-taulukossa olevaan olioon, ja kuten muistamme tilaa ei Reactissa saa muuttaa suoraan!

Kannattaa myös huomata, että uusi olio _changedNote_ on ainoastaan ns [shallow copy](https://en.wikipedia.org/wiki/Object_copying#Shallow_copy), eli uuden olion kenttien arvoina on vanhan olion kenttien arvot. Jos vanhan olion kentät olisivat itsessään olioita, viittaisivat uuden olion kentät samoihin olioihin.


Uusi muistiinpano lähetetään sitten PUT-pyynnön mukana palvelimelle, jossa se korvaa aiemman muistiinpanon.

Takaisinkutsufunktiossa asetataan komponentin _App_ tilaan kaikki vanhat muistiinpanot paitsi muuttuneen, josta tilaan asetetaan palvelimen palauttama versio:

```js
axios.put(url, changedNote)
  .then(response => {
    const nonchangedNotes = this.state.notes.filter(n => n.id !== id)
    this.setState({
      notes: nonchangedNotes.concat(response.data)
    })
  })
```

Ensin muut vanhat muistiinpanot paitsi muutunut otetaan tilasta taulukon muuttujan [filter](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/filter) avulla.

Tila päivitetään [concatenoimalla](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/filter) avulla vanhat muuttumattomat ja palvelimen palauttama muuttunut muistiinpano.

### Kiinteä järjestys

Sovelluksemme toimii, mutta uusi toiminnallisuus vaihtelee ikävästi muistiinpanojen järjestystä. Korjataan asia järjestämällä muistiinpanot aina id-kentän perusteella.

Järjestäminen onnistuu taulukon metodilla [sort](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/sort).

Tehdään järjestäminen metodissa _render_.

```react
render() {
  // ...

  const byId = (note1, note2) => note1.id - note2.id

  return (
    <div>
      // ...

      {notesToShow.sort(byId).map(note =>
        <Note
          key={note.id}
          note={note}
          toggleImportance={this.toggleImportanceOf(note.id)}
        />
      )}

      // ...
    </div>
  )
}
```

Järjestämistä varten on nyt määritelty muuttujaan _byId_ apufunktio, jota kutsutaan ennen kuin _Note_ komponentit generoidaan _map_-metodin avulla:

```js
notesToShow.sort(byId).map.map(note => <Note ... />)
```

## Palvelimen kanssa tapahtuvan kommunikoinnin eristäminen omaan moduuliin

_App_-komponentti alkaa kasvaa uhkaavasti kun myös palvelimen kanssa kommunikointi tapahtuu komponentissa. [Single responsibility](https://en.wikipedia.org/wiki/Single_responsibility_principle) -periaatteen hengessä kommunikointi onkin viisainta eristää omaan [moduuliinsa](#refaktorointia---moduulit).

Luodaan hakemisto _src/services_ ja sinne tiedosto _notes.js_:

```js
import axios from 'axios'
const baseUrl = 'http://localhost:3001/notes'

const getAll = () => {
  return axios.get(baseUrl)
}

const create = (newObject) => {
  return axios.post(baseUrl, newObject)
}

const update = (id, newObject) => {
  return axios.put(`${baseUrl}/${id}`, newObject)
}

export default { getAll, create, update }
```

Moduuli palauttaa nyt olion, jonka kenttinä on kolme muistiinpanojen käsittelyä hoitavaa funktiota. Funktiot palauttavat suoraan axiosin metodien palauttaman promisen.

Komponentti _App_ saa moduuli käyttöön _import_-lauseella

```js
import noteService from './services/notes'

App extends React.Component {
```

moduulin funktioita käytetään importatun muuttujan _noteService_ kautta seuraavasti:

```js
componentWillMount() {
  noteService
    .getAll()
    .then(response => {
      this.setState({notes: response.data})
    })
}

addNote = (e) => {
  // ...
  noteService
    .create(noteObject)
    .then(response => {
      this.setState({
        notes: this.state.notes.concat(response.data),
        new_note: ''
      })
    })
}

toggleImportanceOf = (id) => {
  return () => {
    // ...
    noteService
      .update(id, changedNote)
      .then(response => {
        const notes = this.state.notes.filter(n => n.id !== id)
        this.setState({
          notes: notes.concat(response.data)
        })
      })
  }
}
```

Voisimme viedä ratkaisua vielä askeleen pidemmälle, sillä käyttäessään moduulin funktioita komponentti _App_ saa olion, joka sisältää koko HTTP-pyynnön vastauksen:

```js
noteService
  .getAll()
  .then(response => {
    this.setState({notes: response.data})
  })
```

Eli asia mistä _App_ on kiinnostunut on parametrin kentässä _response.data_.

Moduulia olisi miellyttävämpi käyttää, jos se HTTP-pyynnön vastauksen sijaan palauttaisi suoraan muistiinpanot sisältävän taulukon. Tällöin moduulin käyttö näyttäisi seuraavalta

```js
noteService
  .getAll()
  .then(notes => {
    this.setState({notes: notes})
  })
```

joka voitaisiin [ilmaista hieman tiiviimmin](#kehittyneempi-tapa-olioliteraalien-kirjoittamiseen) seuraavasti:

```js
noteService
  .getAll()
  .then(notes => {
    this.setState({notes})
  })
```

Tämä onnistuu muuttamalla moduulin koodia seuraavasti (koodiin jää ikävästi copy-pastea, emme kuitenkaan nyt välitä siitä):

```js
import axios from 'axios'
const baseUrl = 'http://localhost:3001/notes'

const getAll = () => {
  const request = axios.get(baseUrl)
  return request.then(response => response.data)
}

const create = (newObject) => {
  const request = axios.post(baseUrl, newObject)
  return request.then(response => response.data)
}

const update = (id, newObject) => {
  const request = axios.put(`${baseUrl}/${id}`, newObject)
  return request.then(response => response.data)
}

export default { getAll, create, update }
```

eli enää ei palautetakaan suoraan axiosin palauttamaa promisea, vaan otetaan promise ensin muuttujaan _request_ ja kutsutaan sille metodia _then_:

```js
const getAll = () => {
  const request = axios.get(baseUrl)
  return request.then(response => response.data)
}
```

Täydellisessä muodossa kirjoitettuna viimeinen rivi olisi:

```js
const getAll = () => {
  const request = axios.get(baseUrl)
  return request.then(response => { return response.data })
}
```

Myös nyt funktio _getAll_ palauttaa promisen, sillä promisen metodi _then_ [palauttaa promisen](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/then).

Koska _then_:in parametri palauttaa suoraan arvon _response.data_, on funktion _getAll_ palauttama promise sellainen, että jos HTTP-kutsu onnistuu, antaa promise takaisinkutsulleen HTTP-pyynnön mukana olleen datan, eli se toimii juuri niin kuin haluamme.

Moduulin muutoksen jälkeen täytyy komponentti _App_ muokata _noteService_:n metodien takaisinkutsujen osalta ottamaan huomioon, että ne palauttavat datan suoraan:

```js
class App extends React.component {

  addNote = (e) => {
    // ...
    noteService
      .create(noteObject)
      .then(newNote => {
        this.setState({
          notes: this.state.notes.concat(newNote),
          new_note: ''
        })
      })
  }

  toggleImportanceOf = (id) => {
    return () => {
      // ...

      noteService
        .update(id, changedNote)
        .then(changedNote => {
          const notes = this.state.notes.filter(n => n.id !== id)
          this.setState({
            notes: notes.concat(changedNote)
          })
        })
    }
  }
}
```

Tämä kaikki on hieman monimutkaista ja asian selittäminen varmaan vaan vaikeuttaa sen ymmärtämistä. Internetistä löytyy paljon vaihtelevatasoista materiaalia aiheesta, esim. [tämä](https://javascript.info/promise-chaining).

[You do not know JS](https://github.com/getify/You-Dont-Know-JS) sarjan kirja "Async and performance" selittää asian [hyvin](https://github.com/getify/You-Dont-Know-JS/blob/master/async%20%26%20performance/ch3.md) mutta tarvitsee selitykseen kohtuullisen määrän sivuja.

Promisejen ymmärtäminen on erittäin keskeistä modernissa Javascript-sovelluskehityksessä, joten asiaan kannattaa uhrata kohtuullisessa määrin aikaa.

## Promise ja virheet

Jos sovelluksemme mahdollistaisi muistiinpanojen poistamisen, voisi syntyä tilanne, missä käyttäjä yrittää muuttaa sellaisen muistiinpanon tärkeyttä, joka on jo poistettu järjestelmästä.

Simuloidaan tälläistä tilannetta "kovakoodaamalla" noteServiceen funktioon _getAll_ muistiinpano, jota ei ole todellisuudessa (eli palvelimella) olemassa:

```js
const getAll = () => {
  const request = axios.get(baseUrl)
  const nonExisting = {
    id: 10000,
    content: 'Tätä muistiinpanoa ei ole palvelimelta',
    date: '2017-12-10T17:30:31.098Z',
    important: true
  }
  return request.then(response => response.data.concat(nonExisting))
}
```

Kun valemuistiinpanon tärkeyttä yritetään muuttaa, konsoliin tulee virheilmoitus, joka kertoo palvelimen vastanneen muutosta vastaavaan urliin _/notes/10000_ tehtyyn HTTP PUT -pyyntöön statuskoodilla 404:

![]({{ "/assets/2/14.png" | absolute_url }})

Sovelluksen tulisi pystyä käsittelemään tilanne hallitusti. Jos konsoli ei ole auki, ei käyttäjä huomaa mitään muuta kun sen, että muistiinpanon tärkeys ei vaihdu napin painelusta huolimatta.

Jo [aiemmin](#axios-ja-promiset) mainittiin, että promisella voi olla kolme tilaa. Kun HTTP-pyyntö epäonnistuu, menee pyyntöä vastaava promise tilaan _rejected_. Emme tällä hetkellä käsittele koodissamme promisen epäonnistumista mitenkään.

Promisen epäonnistuminen [käsitellään](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Using_promises) antamalla _then_ metodille parametriksi myös toinen takaisinkutsufunktio, jota kutsutaan siinä tapauksessa jos promise epäonnistuu.

Ehkä yleisempi tapa kuin kahden tapahtumankäsittelijän käyttö on liittää promiseen epäonnistumistilanteen käsittelijä kutsumalla metodia [catch](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise/catch).

Käytännössä virhetilanteen käsittelijän tekisteröiminen tapahtuisi seuraavasti

```js
axios.get('http://example.com/propably_will_fail')
  .then(response => {
    console.log('success!')
  })
  .catch(error => {
    console.log('fail')
  })
```

Jos pyyntö epäonnistuu, kutsutaan _catch_-metodin avulla rekisteröityä käsittelijää.

Metodia _catch_ hyödynnetän usen siten, että se sijoitetaan syvemmälle promiseketjuun.

Kun sovelluksemme tekee HTTP-operaation syntyy oleellisesti ottaen [promiseketju](https://javascript.info/promise-chaining):

```js
axios
  .put(`${baseUrl}/${id}`, newObject)
  .then(response => response.data)
  .then(changedNote => {
    // ...
  })
```

Metodilla _catch_ voidaan määritellä ketjun lopussa käsittelijäfunktio, jota kutsutaan siinä vaiheessa jos mikä tahansa ketjun promisesta epäonnistuu, eli menee tilaan _rejected_:

```js
axios
  .put(`${baseUrl}/${id}`, newObject)
  .then(response => response.data)
  .then(changedNote => {
    // ...
  })
  .catch(error => {
    console.log('fail')
  })
```

Hyödynnetään tätä ominaisuutta, ja sijoitetaan virheenkäsittelijä komponenttiin _App_:

```js
toggleImportanceOf = (id) => {
  return () => {
    const note = this.state.notes.find(n => n.id === id)
    const changedNote = { ...note, important: !note.important }

    noteService
      .update(id, changedNote)
      .then(changedNote => {
        const notes = this.state.notes.filter(n => n.id !== id)
        this.setState({
          notes: notes.concat(changedNote)
        })
      })
      .catch(error => {
        alert(`muistiinpano '${note.content}' on jo valitettavasti poistettu palvelimelta`)
        this.setState({ notes: this.state.notes.filter(n => n.id !== id) })
      })
  }
}
```

Virheilmoitus annetaan vanhan kunnon [alert](https://developer.mozilla.org/en-US/docs/Web/API/Window/alert)-dialogin avulla ja palvelimelta poistettu muistiinpano poistetaan tilasta.

Alertia tuskin kannattaa käyttää todellisissa React-sovelluksissa. Opimme myöhemmin kehittyneempiä menetelmiä käyttäjille tarkoitettujen muistutusten antamiseen.

### Tehtäviä palvelimen tilan päivittämisestä

Tee nyt tehtävät [34-37](../tehtavat#palvelimella-olevan-datan-päivitäminen)

## Tyylien lisääminen

Sovelluksemme ulkoasu on tällä hetkellä hyvin vaatimaton. Osaan 1 liittyvissä [tehtävässä 1](tehtavat/#1-HTML-ja-CSS) oli tarkoitus tutustua Mozillan [CSS-tutoriaaliin](https://developer.mozilla.org/en-US/docs/Learn/Getting_started_with_the_web/CSS_basics).

Katsotaan vielä tämän osan lopussa nopeasti erästä tapaa liittää tyylejä React-sovellukseen. Tapoja on useita ja tulemme tarkastelemaan muita myöhemmin. Liitämme nyt CSS:n sovellukseemme vanhan kansan tapaan yksittäisenä, käsin eli ilman [esiprosessorien](https://developer.mozilla.org/en-US/docs/Glossary/CSS_preprocessor) apua kirjoitettuna tiedostona (tämä ei itseasiassa ole täysin totta, kuten myöhemmin tulemme huomaamaan).

Tehdään sovelluksen hakemistoon _src_ tiedosto _index.css_ ja liitetään se sovellukseen lisäämällä tiedostoon _index.js_ seuraava import:

```js
import './index.css'
```

Lisätään seuraava sääntö tiedostoon _index.css_:

```css
h1 {
  color: green;
}
```

CSS-säännöt koostuvat valitsimesta, eli _selektorista_ ja määrittelystä eli _deklaraatiosta_. Valitsin määrittelee, mihin elementteihin sääntö kohdistuu. Valitsimena on nyt _h1_, eli kaikki sovelluksessa käytetyt _h1_-otsikkotägit.

Määrittelyosa asettaa ominaisuuden _color_, eli fontin värin arvoksi vihreän, eli _green_.

Sääntö voi sisältää mielivaltaisen määrän määrittelyjä. Muutetaan edellistä siten, että tekstistä tulee kursivoitua, eli fontin tyyliksi asetetaan _italics_:

```css
h1 {
  color: green;
  font-style: italic;
}
```

Erilaisia selektoreja eli tapoja valita tyylien kohde on [lukuisia](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Selectors).

Jos haluamme kohdistaa, tyylejä esim. jokaiseen muistiinpanoon voisimme nyt käyttää selektoria _li_, sillä muistiinpanot ovat _li_-tagien sisällä:

```react
const Note = ({ note, toggleImportance}) => {
  const label = note.important ? 'make not important' : 'make important'
  return (
    <li>{note.content} <button onClick={toggleImportance}>{label}</button></li>
  )
}
```

lisätään tyyliedostoon seuraava (koska osaamiseni tyylikkäiden web-sivujen tekemiseen on lähellä nollaa, nyt käytettävissä tyyleissä ole sinänsä mitään järkeä):

```css
li {
  color: grey;
  padding-top: 5px;
  font-size: 15px;
}
```

Tyylien kohdistaminen elementtityypin sijaan on kuitenkin hieman ongelmallista, jos sovelluksessa olisi myös muita _li_-tageja, kaikki saisivat samat tyylit.

Jos haluamme kohdistaa tyylit nimenomaan muistiinpanoihin, on parempi käyttää [class selectoreja](https://developer.mozilla.org/en-US/docs/Web/CSS/Class_selectors).

Normaalissa HTML:ssä luokat määritellään elementtien attribuutin _class_ arvona:

```html
<li class="note">
  tekstiä
</li>
```

Reactissa tulee kuitenkin classin sijaan käyttää attribuuttia [className](https://reactjs.org/docs/dom-elements.html#classname), eli muutetaan komponenttia _Note_ seuraavasti:

```react
const Note = ({ note, toggleImportance}) => {
  const label = note.important ? 'make not important' : 'make important'
  return (
    <li className="note">
      {note.content} <button onClick={toggleImportance}>{label}</button>
    </li>
  )
}
```

Luokkaselektori määritellään syntaksilla _.classname_, eli :

```css
.note {
  color: grey;
  padding-top: 5px;
  font-size: 15px;
}
```

Jos nyt lisäät sovellukseen muita li-elementtejä, ne eivät saa muistiinpanoille määriteltyjä tyylejä.

### Parempi virheilmoitus

Toteutimme äsken olemassaolemattoman muistiinpanon tärkeyden muutokseen liittyvän virheilmoituksen _alert_-metodilla. Toteutetaan se nyt Reactilla omana komponenttinaan.

Komponentti on yksinkertainen:

```react
const Notification = ({ message }) => {
  if (message === null) {
    return null
  }
  return (
    <div className="error">
      {message}
    </div>
  )
}
```

Jos propsin _message_ arvo on _null_ ei renderöidä mitään, muussa tapauksessa renderöidään viesti div-elementtiin. Elementille on liitetty tyylien lisäämistä varten luokka _error_.

Lisätään komponentin _App_ tilaan kenttä _error_ virheviestiä varten, laitetaan kentälle jotain sisältöä, jotta pääsemme heti testaamaan komponenttia:

```js
class App extends React.Component {
  constructor(props) {
    super(props)
    this.state = {
      notes: [],
      new_note: '',
      showAll: true,
      error: 'something went wrong...'
    }
  }
  // ...
}
```

Renderöidään uusi komponentti:

```react
class App extends React.Component {
  render() {
    //...

    return (
      <div>
        <h1>Muistiinpanot</h1>

        <Notification message={this.state.error}/>

        ...
      </div>
    )
  }
}
```

Lisätään sitten virheviestille sopiva tyyli:

```error
.error {
  color: red;
  background: lightgrey;
  font-size: 20px;
  border-style: solid;
  border-radius: 5px;
  padding: 10px;
  margin-bottom: 10px;
}
```

Nyt olemme valmiina lisäämään virheviestin logiikan. Alustetaan virheviesti konstruktorissa arvoon _null_ ja muutetaan metodia _toggleImportanceOf_ seuraavasti:

```js
  toggleImportanceOf = (id) => {
    return () => {
      //...

      noteService
        .update(id, changedNote)
        .then(changedNote => {
          // ...
        })
        .catch(error => {
          this.setState({
            error: `muistiinpano '${note.content}' on jo valitettavasti poistettu palvelimelta`,
            notes: this.state.notes.filter(n => n.id !== id)
          })
          setTimeout(() => {
            this.setState({error: null})
          }, 5000)
        })
    }
  }
```

Eli virheen yhteydessä asetetaan tilan kenttään _error_ sopiva virheviesti. Samalla käynnistetään ajastin, joka asettaa 5 sekunnin kuluttua tilan _error_-kentän arvoksi _null_.


### Loppuhuipennus

Tee nyt tehtävät [38 ja 39](../tehtavat#tyylit)

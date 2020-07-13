```python
import pandas as pd
import re
from collections import Counter
```


```python
pd.options.display.max_colwidth = 100
pd.set_option("display.max_rows", 2000)
```


```python
rcp = pd.read_csv("All_recipes.csv", index_col = 0, encoding = 'utf-8')
```


```python
rcp.info()
```

    <class 'pandas.core.frame.DataFrame'>
    Int64Index: 124581 entries, 0 to 124580
    Data columns (total 24 columns):
     #   Column               Non-Null Count   Dtype  
    ---  ------               --------------   -----  
     0   file_name            124581 non-null  object 
     1   titel_gerecht        124581 non-null  object 
     2   url                  124581 non-null  object 
     3   keuken               92288 non-null   object 
     4   gang                 102918 non-null  object 
     5   tijdsduur            113507 non-null  object 
     6   datum                124581 non-null  object 
     7   intro                124309 non-null  object 
     8   recepttekst          124033 non-null  object 
     9   plaatser_naam        124579 non-null  object 
     10  plaatser_URL         124579 non-null  object 
     11  plaatser_volgers     124581 non-null  int64  
     12  plaatser_recepten    124581 non-null  int64  
     13  plaatser_kookboeken  124581 non-null  int64  
     14  ingrediënten         124581 non-null  object 
     15  tags                 13895 non-null   object 
     16  bekeken              124581 non-null  object 
     17  bewaard              124581 non-null  float64
     18  kookgroepen          124581 non-null  int64  
     19  kookboeken           124581 non-null  int64  
     20  beoordeling_avg      124581 non-null  float64
     21  beoordeling_aantal   124581 non-null  int64  
     22  reactie_aantal       124226 non-null  float64
     23  reacties             124581 non-null  object 
    dtypes: float64(3), int64(6), object(15)
    memory usage: 23.8+ MB



```python
rcp['titel_gerecht'].head()
```




    0    Zoete-aardappelkoekjes van Ottolenghi
    1          Scampi's met schorsenerencrÃ¨me
    2     gekookte muizen met gebakken slakken
    3                     Koffie-advocaatcoupe
    4                          Tomatentapenade
    Name: titel_gerecht, dtype: object




```python
def fix_encoding(s):
    s = s.encode('cp1252', errors="replace").decode('utf-8', errors='replace')
    return s
```


```python
rcp_c = pd.DataFrame()
```


```python
rcp_c["titel_gerecht"] = rcp["titel_gerecht"].apply(lambda s: fix_encoding(s)).str.lower()
```


```python
replace_dict = {'Ingrediënten\s\d{1,2}\spersonen':'','Ingrediënten': ''}


def clean_ingredients(s): 
    
    #replace character multiples with something
    ingr_dict = {
        '\n': ' ',
        'Ã¨': 'e',
        'Ã®':'i',
        'Ã¯':'i',
        'Â': '',
        'Ã©':'e',
        'Â':'x',
        'Ã¡':'x',
        'Ã«':'e',
        'â' : '',
        '¢' : '',
        'Ã¯': 'i',
        'crÃ¨me': 'creme',
        'fraÃ®che':'fraiche',
        '•':''
        }
    for k in ingr_dict:
        s = s.replace(k, ingr_dict[k])
    
    #remove selected characters
    chars = '!@#$~%^&*\'()`_+<>?:.·\\|,";-=/[]{}�'
    for c in s:
        if c in chars:
            s = s.replace(c, " ")    
    
    #remove digits
    digits = re.compile('\d*')
    s = digits.sub('', s)    
    
    #reduces whitespace to one
    multiple_spaces = re.compile('\s+')
    s = multiple_spaces.sub(' ', s)
    s.strip()
    i = s.lower()
    
    return i

stopwords_to_remove = pd.read_csv("ReferenceLists/remove.csv", index_col = 0, encoding = 'utf-8', header=None)
remove_pattern = r'\b(?:{})\b'.format('|'.join(stopwords_to_remove.index))

a_to_remove = pd.read_csv("ReferenceLists/adjectives.csv", index_col = 0, encoding = 'utf-8', header=0)
remove_pattern_a = r'\b(?:{})\b'.format('|'.join(a_to_remove.index))

i_to_remove = pd.read_csv("ReferenceLists/ingredients.csv", index_col = 0, encoding = 'utf-8', header=0)
remove_pattern_i = r'\b(?:{})\b'.format('|'.join(i_to_remove.index))



rcp_c['ingrediënten'] = rcp['ingrediënten'].replace(regex = replace_dict).apply(lambda s: fix_encoding(s)).apply(lambda s: clean_ingredients(s)).str.replace(remove_pattern, '').str.replace(remove_pattern_a, '').str.replace(remove_pattern_i, '')
```


```python
rcp_c[0:500]
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>titel_gerecht</th>
      <th>ingrediënten</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>zoete-aardappelkoekjes van ottolenghi</td>
      <td>geschilde zoete              fijne kristalsuiker   bosui         smaak royaal    bakken   saus...</td>
    </tr>
    <tr>
      <th>1</th>
      <td>scampi's met schorsenerencrème</td>
      <td>scampi  schorseneren        rucolahandje</td>
    </tr>
    <tr>
      <th>2</th>
      <td>gekookte muizen met gebakken slakken</td>
      <td>gekookte  tomaatjes sla komkommer kruidenboter  radijsjes mayonaise tube</td>
    </tr>
    <tr>
      <th>3</th>
      <td>koffie-advocaatcoupe</td>
      <td>hete koffie   bolletjes vanille chocoladeijs  stijfgeslagen   advocaat chocolait chips nestle</td>
    </tr>
    <tr>
      <th>4</th>
      <td>tomatentapenade</td>
      <td>pesto</td>
    </tr>
    <tr>
      <th>5</th>
      <td>kaas-uientaart</td>
      <td>deeg     koude  eidooiers   vulling        ontbijtspek  leerdammer   creme fraiche</td>
    </tr>
    <tr>
      <th>6</th>
      <td>ei-avocadosalade met rauwe ham</td>
      <td>sesamzaad avocado        olijven  piment  rauwe ham veldsla  rucola   dressing     wijnazijn ...</td>
    </tr>
    <tr>
      <th>7</th>
      <td>knolselderij met pesto</td>
      <td>knolselderij gewassen  geschild    zonnebloemolie  selderij   parmezaanse  grofgeraspt  walnot...</td>
    </tr>
    <tr>
      <th>8</th>
      <td>gevuld fruit en groenten</td>
      <td>cherrytomaatjes  huttenkase  mayonaise bosuitje tabasco</td>
    </tr>
    <tr>
      <th>9</th>
      <td>gnocchi met groente</td>
      <td>oudbakken volkorenbrood  hete      wortel venkelknol    cayennepeper paneermeel  pizzatomaten ...</td>
    </tr>
    <tr>
      <th>10</th>
      <td>milkshake met limonadesiroop</td>
      <td></td>
    </tr>
    <tr>
      <th>11</th>
      <td>ijsyoghurt met kiwi</td>
      <td>kiwi    basterdsuiker  magere yoghurt    biologisch ijsblokjes</td>
    </tr>
    <tr>
      <th>12</th>
      <td>yoghurtshake aalbessen</td>
      <td>vruchtensap</td>
    </tr>
    <tr>
      <th>13</th>
      <td>rode bonenpot met shoarmavlees</td>
      <td>kidney bonen    shoarmavlees  boerensoepgroenten  wortel grofgeraspt  tomatensap  brood   oude...</td>
    </tr>
    <tr>
      <th>14</th>
      <td>vegetarische sateballetjes uit de wok</td>
      <td>bosuitjes    tivall vegetarische sateballetjes  peultjes worteltjes    kokosmelk  santen  djint...</td>
    </tr>
    <tr>
      <th>15</th>
      <td>gestoofde sucadelapjes</td>
      <td>sucadelapjesvers      geperste  gesnipperde      gepeld       hot laurierblaadjeswat kruidnage...</td>
    </tr>
    <tr>
      <th>16</th>
      <td>belgische aardappelschotel met kip</td>
      <td>gekookte  gebraden kip  doorregen spek         pers  thijm      reuzel  belangrijk   smaak</td>
    </tr>
    <tr>
      <th>17</th>
      <td>uiensoep met geuze</td>
      <td>geuze   spekblokjes   emmentaler  croutons  fijne kruiden  groentebouillon</td>
    </tr>
    <tr>
      <th>18</th>
      <td>aziatisch gebakken kip uit oven</td>
      <td>kippenbouten    kippenvleugels  puntjes  marinade  komijnpoeder  chilipoeder      gember   rijs...</td>
    </tr>
    <tr>
      <th>19</th>
      <td>fastfood saus</td>
      <td>herbs spices blend   poeder mild  uienpoeder  zeezout eigenlijk uienzout     nergens krijgen  s...</td>
    </tr>
    <tr>
      <th>20</th>
      <td>bananencake</td>
      <td>bananen  pecan noten  honig mix  fijne cake</td>
    </tr>
    <tr>
      <th>21</th>
      <td>srewdriver</td>
      <td>wodka jus dorange</td>
    </tr>
    <tr>
      <th>22</th>
      <td>appelpoffer</td>
      <td>zure appels   basterdsuiker       bakken      kaneelpoeder</td>
    </tr>
    <tr>
      <th>23</th>
      <td>eclairs</td>
      <td>soezendeeg         gezeefde   banketbakkersroom    basterdsuiker  maizena    vanille extract gl...</td>
    </tr>
    <tr>
      <th>24</th>
      <td>sunday brunch hot chorizo en mozzarella stokje</td>
      <td>chorizo    mini mozzarella bolletjes plat gedrukt stokbrood  keuze   baby spinazie</td>
    </tr>
    <tr>
      <th>25</th>
      <td>venkelschotel met aardappelpuree</td>
      <td>venkelknollen  geschilde            pikante       aroma</td>
    </tr>
    <tr>
      <th>26</th>
      <td>pistache dream (cocktail)</td>
      <td></td>
    </tr>
    <tr>
      <th>27</th>
      <td>italiaanse eieren</td>
      <td>salami</td>
    </tr>
    <tr>
      <th>28</th>
      <td>sate van varkensvlees (sate babi)</td>
      <td>varkenshaasjes  sambal  japanse ongesuikerde       gemberpoeder  ketoembar</td>
    </tr>
    <tr>
      <th>29</th>
      <td>polvorones (anijskoekjes) *</td>
      <td>bakpoeder    vanillesuiker  anijszaadjes  margarine losgeklopt  poedersuiker</td>
    </tr>
    <tr>
      <th>30</th>
      <td>kleurrijke risotto</td>
      <td>stegeman licht pittige chorizo    kleintje uitjes        tuinbonen  kastanjechampignons  cherr...</td>
    </tr>
    <tr>
      <th>31</th>
      <td>rode kipcurry met &amp;lparspits&amp;rparkool en champignons</td>
      <td>spitskool          lenteuitjes    kokosmelk    curry pasta kaffir lime leaves  vissaus      ...</td>
    </tr>
    <tr>
      <th>32</th>
      <td>rundergehakt met garnalen en ansjovis</td>
      <td>rundergehakt  hollandse garnalen koffie  uitgelekte ansjovisfilets</td>
    </tr>
    <tr>
      <th>33</th>
      <td>mexicaanse ovenschotel</td>
      <td>vind     smeuiger  puur rundergehakt uiten   chilipoeder  chilibonen  mais  salsa saus  c...</td>
    </tr>
    <tr>
      <th>34</th>
      <td>hoemoetdat? darmen</td>
      <td>voorbewerking  darmendarmen     gesmoord gestoofd  gebakken   varkensdarmen   omhulsel gebruikt...</td>
    </tr>
    <tr>
      <th>35</th>
      <td>ons favoriete fonduegerecht:</td>
      <td>bier sausje  smaak fonduepan   heet</td>
    </tr>
    <tr>
      <th>36</th>
      <td>schelvis met amandelen</td>
      <td>schelvissen      beeje   amandelschilfers sap  citroenen</td>
    </tr>
    <tr>
      <th>37</th>
      <td>gegratineerde mosselen met tomaat en venkel</td>
      <td>mosselen fijngesnipperd seldertakje fijngesnipperde   peterselietakje   dillezaad      molen  ...</td>
    </tr>
    <tr>
      <th>38</th>
      <td>goofy's soepstengels</td>
      <td>soepstengels   ontbijtspek</td>
    </tr>
    <tr>
      <th>39</th>
      <td>mousse au chocolat (chocolademousse)</td>
      <td>bittere couverture  chocolade    cognac  gepasteuriseerd eiwit</td>
    </tr>
    <tr>
      <th>40</th>
      <td>monchou gebakjes</td>
      <td>monchou    variatievla  stoofpeertjes campina   basterdsuiker gebaksschelpen jos poell gestoof...</td>
    </tr>
    <tr>
      <th>41</th>
      <td>varkensgebraad met venkel-sinaasappelsausje</td>
      <td>varkensgebraad venkel sap sinaasappelen sjalot       kalfsfond   instantsausbinder korianderbo...</td>
    </tr>
    <tr>
      <th>42</th>
      <td>soep onder een dekseltje</td>
      <td>rijst zeezout    port   heldere ossetaartsoep  bladerdeeg   tauge</td>
    </tr>
    <tr>
      <th>43</th>
      <td>appeltaart met een vleugje rum</td>
      <td>deeg  zelfrijzend bakmeel  roomboter  lichtbruine basterdsuiker  vanille     vulling  zure appe...</td>
    </tr>
    <tr>
      <th>44</th>
      <td>pinda bravoe</td>
      <td>kip  rundvlees  zoutvlees   sjalotjes pimentkorrels bouillonblokjes  pindakaas   halfrijpe   s...</td>
    </tr>
    <tr>
      <th>45</th>
      <td>gerookte schelvispat�?</td>
      <td>schelvis    margarine        aromat   sla   garnering    garnering  volkoren broden  korst  r...</td>
    </tr>
    <tr>
      <th>46</th>
      <td>gevulde tomaten</td>
      <td>vleestomaten mozzarella balletjes  paneermeel bosuitjes     mais     geknipte   griekse kruiden...</td>
    </tr>
    <tr>
      <th>47</th>
      <td>grapefruitjam</td>
      <td>roze grapefruits ananas    citroenzuur   geleisuiker speciaal</td>
    </tr>
    <tr>
      <th>48</th>
      <td>peperkoekster.</td>
      <td>zachte   margarine      bakvorm paneermeel   bakvorm     tarwebloem    kruidnagelpoeder  bakpo...</td>
    </tr>
    <tr>
      <th>49</th>
      <td>koffie met advokaat</td>
      <td>koffie advokaat</td>
    </tr>
    <tr>
      <th>50</th>
      <td>aardappelsalade met tonijn</td>
      <td>tonijn    yoghurt  mayonaise  zure    bieslooksprietjes  petersellie</td>
    </tr>
    <tr>
      <th>51</th>
      <td>bietenbrownies</td>
      <td>bieten   zelfrijzend bakmeel   basterdsuiker  pure chocolade  cacao  vanillesuiker</td>
    </tr>
    <tr>
      <th>52</th>
      <td>parfait van bosvruchten</td>
      <td>eierdooiers    bosbessencoulis  rodebessencoulis  frambozenpuree   bosvrucht  vruchtenextract</td>
    </tr>
    <tr>
      <th>53</th>
      <td>lasagna van shirley</td>
      <td>rundergehakt  ontbijtspek          blad laurier  oregano      parmesaanse         lasagna  honig</td>
    </tr>
    <tr>
      <th>54</th>
      <td>teriyaki zalmspiesjes</td>
      <td>spiesjes  zalmfilet satestokjes   teriyaki marinade  fijne tafelsuiker ertlrprl mirin japanse...</td>
    </tr>
    <tr>
      <th>55</th>
      <td>nederhorst den bergse  kalfsoesterrolletjes</td>
      <td>kalfsoesters   gezouten spek   noord hollandse belegen   knoflookbol sjalot    wortel        ...</td>
    </tr>
    <tr>
      <th>56</th>
      <td>pasta met spaanse invloeden.</td>
      <td>tomato frito      katenspek  rundergehakt    rioja     slankroom walnoten  spiraalmacaroni zoe...</td>
    </tr>
    <tr>
      <th>57</th>
      <td>ultieme kaastosti</td>
      <td>casinoboterhammen  roomboter  parrano strooikaas  jonge   emmentaler  buffelmozzarella  boursi...</td>
    </tr>
    <tr>
      <th>58</th>
      <td>pruimenlikeur</td>
      <td></td>
    </tr>
    <tr>
      <th>59</th>
      <td>eendeborstfilet met portosaus</td>
      <td>eendeborstfilets porto  kruidenbouillon   wildfond  tuinkruiden</td>
    </tr>
    <tr>
      <th>60</th>
      <td>burger van kipfilet en garnalen</td>
      <td>sjalotje    hollandse garnalen eierdooier    tomatenketchup    paneermeel      ijsbergsla  bals...</td>
    </tr>
    <tr>
      <th>61</th>
      <td>kaaspate met noten</td>
      <td>snee volkorenbrood  korst  magere   sojamelk  walnoten  pecannoten     geraspt    uitgeperst   ...</td>
    </tr>
    <tr>
      <th>62</th>
      <td>aardbeienbowl</td>
      <td>aardbeien  cointreau  poedersuiker fles   fles mousserende   koel</td>
    </tr>
    <tr>
      <th>63</th>
      <td>tagliatelle caprse-style</td>
      <td>à  mozzarella     geperst         tagliatelle varkensoesters  kalfsoesters</td>
    </tr>
    <tr>
      <th>64</th>
      <td>gezonde en gevulde loempia</td>
      <td>maggi roerbaknoedels mediterrane kip     groente  loempia</td>
    </tr>
    <tr>
      <th>65</th>
      <td>valess camembert met tagliatelle en sperziebonen</td>
      <td>valess camambert filets  tagliatelle     sperziebonen panklaar        grove   groentebouillon</td>
    </tr>
    <tr>
      <th>66</th>
      <td>bruine bonen met rijst</td>
      <td>winterwortel  tomaat  bonen selderij    madam jaenet surinaamse   bouillonblokje kruiden  ...</td>
    </tr>
    <tr>
      <th>67</th>
      <td>spaghetti met garnalen en rucola</td>
      <td>spaghetti     geperst   pepers zaad verwijderd   trostomaat ontveld   sap  rasp     gekookte g...</td>
    </tr>
    <tr>
      <th>68</th>
      <td>wiener melange</td>
      <td>eierdooiers    sterk koffie extract    schep</td>
    </tr>
    <tr>
      <th>69</th>
      <td>boerenkwark/frambozen bavaroise</td>
      <td>frambozen  kristalsuiker    boerenkwark  gelatine  aardbeiensiroop</td>
    </tr>
    <tr>
      <th>70</th>
      <td>vruchtensalade</td>
      <td>lychees  siroop  galia meloen  aardbeien bolletjes stemgember  gembersiroop</td>
    </tr>
    <tr>
      <th>71</th>
      <td>pita speciaal</td>
      <td>pita broodjes jonge jong belegen  knoflookpoeder rozemarijn</td>
    </tr>
    <tr>
      <th>72</th>
      <td>mi­ne­stro­ne met gar­na­len</td>
      <td>winterpeen   sperziebonen  traditionele     kraanwater visbouillontabletten  cannellinibonen ...</td>
    </tr>
    <tr>
      <th>73</th>
      <td>maple-syrup met pecannoten</td>
      <td>pecannoten   maple syrup</td>
    </tr>
    <tr>
      <th>74</th>
      <td>het calculettenhapje</td>
      <td>varkensvlees taco  ijsbergsla  paddestoelen   sperzibonen kruiden laos ketoembar roosmaijn     ...</td>
    </tr>
    <tr>
      <th>75</th>
      <td>varkensfilet met sla</td>
      <td>komkommer   c  braad vloeibaar  c bistro minikriel à  varkensfiletlapjes    c ijsbergsla  tuink...</td>
    </tr>
    <tr>
      <th>76</th>
      <td>krakend krokante malse kippenpoten</td>
      <td>kippenpoten  kerriepoeder   chilipoeder   pikant  knoflookpoeder      zeezout    rozemarijn  t...</td>
    </tr>
    <tr>
      <th>77</th>
      <td>tongfilet met rabarber en dragon</td>
      <td>tongfilets   dragonblaadjes garnering     saus  rabarber        rietsuiker  kaneelpoeder    dra...</td>
    </tr>
    <tr>
      <th>78</th>
      <td>broccoli soep &amp;lparheel dik&amp;comma heel lekker&amp;rpar</td>
      <td>winterwortel    broccoli  bouillon groenten  kruiden zongedroogde tomaatjes       pesto  ita...</td>
    </tr>
    <tr>
      <th>79</th>
      <td>gergillde kip met paprika-garnalensaus</td>
      <td>kip    kipkruiden  gepelde hollandse garnalen   roomsaus   tabasco</td>
    </tr>
    <tr>
      <th>80</th>
      <td>ham-kaassoep</td>
      <td>kippenbouillon zelfgemaakt        koffieroom   oude goudse   gekookte ham</td>
    </tr>
    <tr>
      <th>81</th>
      <td>geitenkaasquiche met noten</td>
      <td>bladerdeeg  fijngemalen amandelen   walnoten    crème fraiche   zacht geitenkaasje     zeezout</td>
    </tr>
    <tr>
      <th>82</th>
      <td>spekpoffers</td>
      <td>gesn doorregen spek    aardappel puree vet      bakken</td>
    </tr>
    <tr>
      <th>83</th>
      <td>gebakken garnalen met ananas</td>
      <td>rauwe garnalen  ananas  zeekraal lenteuitjes    pepertje  wokolie soya saus</td>
    </tr>
    <tr>
      <th>84</th>
      <td>thaise rode curry met zalm&amp;period</td>
      <td>ingrediënten  zonnebloemolie à  thaise  currypasta    kokosmelk   zalmfilet     schoongemaakte ...</td>
    </tr>
    <tr>
      <th>85</th>
      <td>paddestoelenrisotto</td>
      <td>lente uitjes wit  vieren groen  ringen     gemengde paddestoelen    risottorijst arborio  warme...</td>
    </tr>
    <tr>
      <th>86</th>
      <td>citroen-knoflooksoep met garnalen</td>
      <td>sjalotjes    garnering limoen  pijnboompitten   spaanse   garnalen      schaal       groenten...</td>
    </tr>
    <tr>
      <th>87</th>
      <td>broodpudding op grootmoeders wijze</td>
      <td>halfvolle     vanillesuiker  becel bakken braden vloeibaar   invetten   bakvorm  oud brood  ko...</td>
    </tr>
    <tr>
      <th>88</th>
      <td>thaise kippensoep met noedels</td>
      <td>kipfilets       plantaardige       kurkuma  chilipoeder    smaak  kokoscrème  warme kippenbouil...</td>
    </tr>
    <tr>
      <th>89</th>
      <td>lamsrollade met kaneel</td>
      <td>lamsrollade     rozemarijn     krieltjes</td>
    </tr>
    <tr>
      <th>90</th>
      <td>kwarkmousse met gember</td>
      <td>chocolade    walnoten grof      bastedsuiker  franse kwark  gembersiroop bolletjes gember  si...</td>
    </tr>
    <tr>
      <th>91</th>
      <td>witte chocomousse met hazelnootlikeur</td>
      <td>chocolade     poedersuiker  hazelnootlikeur   likeur  keuze      cacao   bestrooien</td>
    </tr>
    <tr>
      <th>92</th>
      <td>courgettes met ricotta</td>
      <td>courgettes      ricotta   parmezaan   pijnboompitten</td>
    </tr>
    <tr>
      <th>93</th>
      <td>zomerstamppotje</td>
      <td>aardappels        bladspinazie  rucola  chettytomaatjes geroosterd   grill  zongedroogde tomaa...</td>
    </tr>
    <tr>
      <th>94</th>
      <td>creme catalana</td>
      <td>waarvan   eigeel  gebruikt      maizena  kaneelstokje</td>
    </tr>
    <tr>
      <th>95</th>
      <td>crempog</td>
      <td>karnemelk        bakpoeder</td>
    </tr>
    <tr>
      <th>96</th>
      <td>green fizz</td>
      <td>gin  frisdrank</td>
    </tr>
    <tr>
      <th>97</th>
      <td>nacht van de hawaiaanse kip</td>
      <td>builtjes rijst  chicken tonight hawai  kalkoenfilet</td>
    </tr>
    <tr>
      <th>98</th>
      <td>bas rond de lapin a la gaillarde</td>
      <td>ganzen eenden  varkensvet bout    elkaar    konijn   sjalotjes    eekhoorntjesbrood</td>
    </tr>
    <tr>
      <th>99</th>
      <td>kokos vs. melk</td>
      <td></td>
    </tr>
    <tr>
      <th>100</th>
      <td>tropical</td>
      <td>vruchtensap</td>
    </tr>
    <tr>
      <th>101</th>
      <td>zalmschuitjes met groene asperges en ricotta</td>
      <td>bladerdeeg  ricotta    aspergetips        zalm</td>
    </tr>
    <tr>
      <th>102</th>
      <td>torta alla crema</td>
      <td>deeg    kristalsuiker      bakpoeder  schil        banketbakkersroom        kristalsuiker  sc...</td>
    </tr>
    <tr>
      <th>103</th>
      <td>spaghetti met garnalen in tomaten-roomsaus</td>
      <td>spaghetti  gepelde   garnalen         oregano  zeezout   tabasco</td>
    </tr>
    <tr>
      <th>104</th>
      <td>zuurkool goulash</td>
      <td>varkenslappen   randje vet  zonnenbloemolie   geplette knoflooktenen       milde      kummel k...</td>
    </tr>
    <tr>
      <th>105</th>
      <td>tagine van kalkoen met zoete aardappelen</td>
      <td>kalkoenfilets á zoete  dadels  pit           gemberpoeder    komijn  kurkuma   smaak</td>
    </tr>
    <tr>
      <th>106</th>
      <td>honing dijon ham en emmenthaler kip</td>
      <td>kipfilets      dijon      ham   goede kwaliteit  emmenthaler     smaak</td>
    </tr>
    <tr>
      <th>107</th>
      <td>een geweldige risotto gemaakt van bietjes en geitenkaas</td>
      <td>groentebouillon  gekookte  bietjes     vergine       geperst     arboriorijst     zachte geite...</td>
    </tr>
    <tr>
      <th>108</th>
      <td>broccolistamppot met ham&amp;comma ei en ui</td>
      <td>kruimige   broccoli   hamblokjes    sambal badjak   manis klont</td>
    </tr>
    <tr>
      <th>109</th>
      <td>wijting met paprika en venkel</td>
      <td>wijtingfilet  venkel knollen         cajunkruiden</td>
    </tr>
    <tr>
      <th>110</th>
      <td>paradiso(aperitief)</td>
      <td>gin abricot brandy appelsiensap</td>
    </tr>
    <tr>
      <th>111</th>
      <td>fallen angel</td>
      <td>gin vruchtensap</td>
    </tr>
    <tr>
      <th>112</th>
      <td>italiaanse doperwtjes</td>
      <td>doperwten     pesto    parmezaanse  geraspt</td>
    </tr>
    <tr>
      <th>113</th>
      <td>rodekool met appeltjes</td>
      <td>eko rodekool   port ± kruidnagels kaneelstokje   rozijnen appels  aardappelmeel</td>
    </tr>
    <tr>
      <th>114</th>
      <td>snelle en lekkere courgettesoep</td>
      <td>courgette  groentenbouillon      winterwortel geschrapt  gepeld  chilipoeder   komijn djintan ...</td>
    </tr>
    <tr>
      <th>115</th>
      <td>linda s najaars risotto</td>
      <td>risottorijst  heet  runderbouillonblokje   geperst  magere spekreepjes  kastanje   boerenkool</td>
    </tr>
    <tr>
      <th>116</th>
      <td>franse tomatentaart met gruyère</td>
      <td>bladerdeeg trostomaten   stevige rijpe   gruyère  comtékaas dijonmosterd provençaalse kruiden ...</td>
    </tr>
    <tr>
      <th>117</th>
      <td>vlechtbroodje hawaii</td>
      <td>vlechtbroodje schijf ananas    kwart  tuinkers halvarine  smeerworst</td>
    </tr>
    <tr>
      <th>118</th>
      <td>keylime-pie</td>
      <td>invetten  country biscuits limoenen</td>
    </tr>
    <tr>
      <th>119</th>
      <td>oranjekoek</td>
      <td>zelfrijzend bakmeel   basterd  gtam       anijszaad  schil   sinasappel  oranjesnippers  amand...</td>
    </tr>
    <tr>
      <th>120</th>
      <td>sayur asem</td>
      <td>grof  kool  boontjes kwart    lombok bouillon  gesn   knofl  trassi  laos  asemwater  gula dja...</td>
    </tr>
    <tr>
      <th>121</th>
      <td>italiaans borrelhapje</td>
      <td>pizzadeeg  napoltana saus  tonijn  olijven     italiaanse kruiden</td>
    </tr>
    <tr>
      <th>122</th>
      <td>mini angusburgertjes</td>
      <td>mini hamburgerbroodjes  jumbo mini angusburgertjes  jumbo  mayonaise  ketchup      komijnzaadpo...</td>
    </tr>
    <tr>
      <th>123</th>
      <td>meatydamia nwankwo soep</td>
      <td>lamslappen      elipsvormige      chilipoeder  oregano      smaak  kabeljauwhaas graten verwij...</td>
    </tr>
    <tr>
      <th>124</th>
      <td>ei  van het huis</td>
      <td>uiteraard    spek   soepgroente</td>
    </tr>
    <tr>
      <th>125</th>
      <td>italiaanse groentepizza</td>
      <td>hak groenteschotel  spaghetti  gekoeld pizzadeeg    ringen       olijven  pit  mortadella ital...</td>
    </tr>
    <tr>
      <th>126</th>
      <td>gegratineerde groenten</td>
      <td>broccoli  bloemkool wortels  diepvrieserwten    saus  zonnebloemolie  tarwebloem    groentenbo...</td>
    </tr>
    <tr>
      <th>127</th>
      <td>appel kokoscake</td>
      <td>amandelmeel  kokosmeel   roomboter kamertemperatuur    kokosvet  vierge  rijstmaltstroop  bev...</td>
    </tr>
    <tr>
      <th>128</th>
      <td>oesterzwammen in citroenboter</td>
      <td>fijne kervel      citroentijm  zachte   paneermeel       oesterzwammen   partjes   schil  groe...</td>
    </tr>
    <tr>
      <th>129</th>
      <td>groentecurry met kalkoen en gamba's</td>
      <td>borstfilets  kalkoen  gekookte gamba    sjalotten   gesnipperde sjalot  groentereepjes   tomate...</td>
    </tr>
    <tr>
      <th>130</th>
      <td>varkensreepjes met groenten</td>
      <td>varkensvlees    gele  courgette lenteuitjes          gemberpoeder</td>
    </tr>
    <tr>
      <th>131</th>
      <td>perziken met amandelvulling</td>
      <td>blanke stroop      rijpe perziken gehalveerd  ontpit  amarettikoekjes verkruimeld   amandelen ...</td>
    </tr>
    <tr>
      <th>132</th>
      <td>marshmellow fluff op beshuit</td>
      <td>dextrose       poedersuiker  vanille extract beschuitjes</td>
    </tr>
    <tr>
      <th>133</th>
      <td>mediterrane kiprolletjes marsala</td>
      <td>kipfilets  meesterlyck rauwe ham verstegen spicemix del mondo  zeezout italiaans zeezout  zwar...</td>
    </tr>
    <tr>
      <th>134</th>
      <td>broodje pindakaas</td>
      <td>soorten pindakaas     brood wit  bruin  zachte  harde broodjes  gebruikt besmeer  broodjes  p...</td>
    </tr>
    <tr>
      <th>135</th>
      <td>appelkruimel</td>
      <td>appels      basterdsuiker  zelfr bakmeel</td>
    </tr>
    <tr>
      <th>136</th>
      <td>spinazie met zalmroosjes</td>
      <td>friseesla  frambozenazijn       jonge hollandse spinazie  zalm</td>
    </tr>
    <tr>
      <th>137</th>
      <td>portabello met garnalen</td>
      <td>personen portabello       gedeelte sjalot     hollandse garnalen    molen  tuinkruidendrinkbo...</td>
    </tr>
    <tr>
      <th>138</th>
      <td>[b]white spider[/b]</td>
      <td>wodka</td>
    </tr>
    <tr>
      <th>139</th>
      <td>amandel-kaasrol</td>
      <td>amandelen  belegen   roomkaas        wocestershire sauce</td>
    </tr>
    <tr>
      <th>140</th>
      <td>zeebaars met venkel, dille, citroen en tomaat</td>
      <td>venkelknollen       ringen    mooie  zeebaarzen       oregano    trostomaatjes  dille mooie  v...</td>
    </tr>
    <tr>
      <th>141</th>
      <td>chilli-olie</td>
      <td>spaanse pepers  dunner  heter</td>
    </tr>
    <tr>
      <th>142</th>
      <td>hot artichoke dip</td>
      <td>bevroren spinazie  artichokeharten afgegoten  kleingesneden   mozzarella    geperst  mayonaise...</td>
    </tr>
    <tr>
      <th>143</th>
      <td>knoflooksaus</td>
      <td>oud witbrood  amandelschaafsel     appelcider   wijnazijn</td>
    </tr>
    <tr>
      <th>144</th>
      <td>cappuccinocrème</td>
      <td>gelatine  sterke koffie  vanillesuiker</td>
    </tr>
    <tr>
      <th>145</th>
      <td>wat is baksoda?</td>
      <td></td>
    </tr>
    <tr>
      <th>146</th>
      <td>gegrilde avocado met tartaar van tonijn</td>
      <td>tonijnfilet bosjes lente uitjes    chilipeper   sap  schil    zeezout   rijpe avocado      g...</td>
    </tr>
    <tr>
      <th>147</th>
      <td>kip in het pannetje</td>
      <td>panklare kip  kipkruiden  rook  katenspek      margarine    braadproduct     worteltjes       r...</td>
    </tr>
    <tr>
      <th>148</th>
      <td>pompoensoep</td>
      <td>mooie ronde pompoen</td>
    </tr>
    <tr>
      <th>149</th>
      <td>groenten gerecht</td>
      <td>geschrapte wortelen    doperwten    sperzie bonen   geschild</td>
    </tr>
    <tr>
      <th>150</th>
      <td>dropstaafjes</td>
      <td>pernod</td>
    </tr>
    <tr>
      <th>151</th>
      <td>amerikaans: gamba's in preiblad</td>
      <td>preibladeren gamba  sap  limoenen     saus       gemberwortel   kokosmelk lente uitjes   ringe...</td>
    </tr>
    <tr>
      <th>152</th>
      <td>groentesoep</td>
      <td>bouillon  soepgroenten  vermicelli  aangemaakt runder  kalfsgehakt</td>
    </tr>
    <tr>
      <th>153</th>
      <td>gevulde appeltaart</td>
      <td>granny smith  roijnen  pecannoten    dadels     basterdsuiker   citroenschil  sinaasappel marm...</td>
    </tr>
    <tr>
      <th>154</th>
      <td>spiesen van gamba's met walnotenpesto</td>
      <td>ongepelde gamba   walnoten rl          parmezaanse     walnotenolie</td>
    </tr>
    <tr>
      <th>155</th>
      <td>panino italiano</td>
      <td>snee  gebakken volkoren brood mini bolletjes mozzarella gehalveerd    italiaanse kruiden  smaa...</td>
    </tr>
    <tr>
      <th>156</th>
      <td>gestoofd konijn</td>
      <td>tam konijn  ±    verdeeld     molen    kleingesneden worteltjes     ontveld           laurierbl...</td>
    </tr>
    <tr>
      <th>157</th>
      <td>spaghetti</td>
      <td>spagetti tomatensaus italiaanse keukenkruiden knoflookpoeder   tonijn cremefreche</td>
    </tr>
    <tr>
      <th>158</th>
      <td>varkenskoteletten op z'n ardeens</td>
      <td>varkenkoteletten     sjalotten        parika  ardeense ham    dragon  kervel</td>
    </tr>
    <tr>
      <th>159</th>
      <td>farizeeër (koffie met rum).</td>
      <td>koffie rum</td>
    </tr>
    <tr>
      <th>160</th>
      <td>mijn oma's kip</td>
      <td>gare soepkip    mayonaise     rijst</td>
    </tr>
    <tr>
      <th>161</th>
      <td>pasta con pesto met bleekselderij-salade</td>
      <td>pesto  hieronder  pasta  courgette        italiaanse kruiden  zonnebloempitten   pompoenpitten...</td>
    </tr>
    <tr>
      <th>162</th>
      <td>mediterrane kiprolletjes</td>
      <td>margarine kiprolletjes ah  aardappelpartjes  schil courgette    olijven  pit fles sun dried ...</td>
    </tr>
    <tr>
      <th>163</th>
      <td>kipschnitzel hawaii</td>
      <td>kipschnitzels   kerrie  gemberpoeder  ananas  belegen goudse   rauwe ham</td>
    </tr>
    <tr>
      <th>164</th>
      <td>ratatouille</td>
      <td>aubergine  courgette   gele     knoflooktenen      rozemarijn</td>
    </tr>
    <tr>
      <th>165</th>
      <td>romige hutspot uit de oven met een tussenlaag pittig gekruid gehakt</td>
      <td>kruimige aardappels geschild  vieren   winterpeen geschild      geschild    zonnebloemolie  ro...</td>
    </tr>
    <tr>
      <th>166</th>
      <td>tomaten gevuld met amandelrijst</td>
      <td>vleestomaten         fijngesnip grrijst  snelkook adl groentenbouillon  geweekte blanke rozijne...</td>
    </tr>
    <tr>
      <th>167</th>
      <td>omelet foe-yong-hai</td>
      <td>champignon  tauge   chilisaus  gemberpoeder stronk      kool</td>
    </tr>
    <tr>
      <th>168</th>
      <td>pesto uit de losse hand</td>
      <td>handjes pijnboompitten    kwart  rucola    parmezaanse  drie   olijven natuurlijk  pit  scheut...</td>
    </tr>
    <tr>
      <th>169</th>
      <td>snelle champignonsalade</td>
      <td>knoopchampignons dressing     wijnazijn  mosterdpoeder    kruiden</td>
    </tr>
    <tr>
      <th>170</th>
      <td>pizzettes</td>
      <td>rood  groen  kleingesneden vlees rookworst  worst  ham   uitgeperst  gemengde groente wortel ...</td>
    </tr>
    <tr>
      <th>171</th>
      <td>gewokte groentecurry</td>
      <td>vastkokende  winterwortels   kool gepelde amandelen conimex wokolie  conimex kruidenpasta  kor...</td>
    </tr>
    <tr>
      <th>172</th>
      <td>pasta e fagioli</td>
      <td>etl   wortel                  gare  kidneybonen  kippenbouillon  pasta ruote     parmezaanse</td>
    </tr>
    <tr>
      <th>173</th>
      <td>italiaans fantasiedessert</td>
      <td>eierdooiers eiwitten  kristalsuiker  gelatinepoeder  vanillesuiker   couverture    italiaanse b...</td>
    </tr>
    <tr>
      <th>174</th>
      <td>pompoen met basilicum</td>
      <td>winterpompoen      boulion         belegen edammer</td>
    </tr>
    <tr>
      <th>175</th>
      <td>hollandse erwtensoep</td>
      <td>spliterwten  rookspek   hamschijf  laurierblaadjes  peperkorrels  preien aardappels winterwort...</td>
    </tr>
    <tr>
      <th>176</th>
      <td>sinaasappel-kwark toettie</td>
      <td>perssinasappelen  agar agar   heldere   magere kwark  vanillesuiker</td>
    </tr>
    <tr>
      <th>177</th>
      <td>sneeuwmannetje</td>
      <td>kokerdropjes   engelse drop dropveters  vanilleroomijs discopillen bakproduct    winegums lolli...</td>
    </tr>
    <tr>
      <th>178</th>
      <td>avocadococktail</td>
      <td>kippenbouillontabletten cayennepeper mayonaise ijsbergsla  sjalotje  zure  avocado     walnoten</td>
    </tr>
    <tr>
      <th>179</th>
      <td>vleessaus uit bologna</td>
      <td>rundgehakt     salami worstje   wortel bouillon selderiestengel      gepelde</td>
    </tr>
    <tr>
      <th>180</th>
      <td>mikkemannen</td>
      <td>soja     geklutst    bestrijken vlak   afbakken   gist  sojaboter  suikerparels</td>
    </tr>
    <tr>
      <th>181</th>
      <td>aardappelpannetje met een restje gebraden vlees &amp;period&amp;period&amp;period&amp;period&amp;period&amp;periodkartof...</td>
      <td>vastkokende     gebraden vlees  mager  spekreepjes</td>
    </tr>
    <tr>
      <th>182</th>
      <td>risotto met tuinkruiden</td>
      <td>sjalot  dille  rozemarijn  salie  dragon  munt  oregano  spinazie   arboriorijst       kippenbo...</td>
    </tr>
    <tr>
      <th>183</th>
      <td>chocoladepudding met croissants</td>
      <td>croissants  chocoladepasta      pure chocolade  fijnesuiker</td>
    </tr>
    <tr>
      <th>184</th>
      <td>italiaanse kabeljauwschotel</td>
      <td>kabeljauwfilet  parmezaanse    gekleurde lint macaroni  creme fraiche      italiaanse kruiden...</td>
    </tr>
    <tr>
      <th>185</th>
      <td>mangodream</td>
      <td>vruchtensap</td>
    </tr>
    <tr>
      <th>186</th>
      <td>sap van aardperen</td>
      <td>uitkiezen kies stevige aardperen   onbeschadigde schil</td>
    </tr>
    <tr>
      <th>187</th>
      <td>aardbeienflutes</td>
      <td>aardbeien  suikersiroop sap     aardbeienlikeur  frambozenlikeur</td>
    </tr>
    <tr>
      <th>188</th>
      <td>tia maria</td>
      <td>jenever</td>
    </tr>
    <tr>
      <th>189</th>
      <td>mexicaanse roerbakschotel</td>
      <td>gekruid reepjesvlees  uienringen  paprikablokjes rood  courgette      smaak   boerenkaas</td>
    </tr>
    <tr>
      <th>190</th>
      <td>gebakken yoghurt met gemengde bessen</td>
      <td>geklopte      griekse yoghurt  vanille essence  maïsmeel  tarwe   gemengde bessen  geroosterde ...</td>
    </tr>
    <tr>
      <th>191</th>
      <td>oosters gekruide kipspiesjes in ontbijtspek</td>
      <td>±    ontbijtspek     oosterse kruiden  ketoembar djahé laos djintan pittig sausje  zoet zure ch...</td>
    </tr>
    <tr>
      <th>192</th>
      <td>gevulde hele kip met spek, sjalotten en appel</td>
      <td>kip     ontbijtspekreepjes grof  sjalotten  appel geschild zonderklokhuis  partjes       holla...</td>
    </tr>
    <tr>
      <th>193</th>
      <td>amuse ´gazpacho van paprika, tomaat en feta´</td>
      <td>witbrood     balsamico    tabasco  feta       provencaalse kruiden</td>
    </tr>
    <tr>
      <th>194</th>
      <td>de aller-lekkerste pompoensoep</td>
      <td>biologische pompoen     winterwortel  geschild            masala laurierblaadjes  komijn djinte...</td>
    </tr>
    <tr>
      <th>195</th>
      <td>ovenschotel met raapstelen</td>
      <td>raapstelen  margarine  cottage cheese huttenkase   parmezaanse</td>
    </tr>
    <tr>
      <th>196</th>
      <td>ljus grundsas</td>
      <td>margarine   kippe  vleesbouillonblokje  heet</td>
    </tr>
    <tr>
      <th>197</th>
      <td>naanbroodjes met kiptikka</td>
      <td>gemberwortel     limoenen    ketoembar   komijn djinten  chilipoeder      griekse yoghur...</td>
    </tr>
    <tr>
      <th>198</th>
      <td>broodje "alles d'r op en alles d'r aan&amp;period</td>
      <td>ciabatta   broodje  smaak  trostomaat    radijsjes    augurk   hardgekookt  afgekoeld    jong ...</td>
    </tr>
    <tr>
      <th>199</th>
      <td>abrikozencarre</td>
      <td>velletjes diepvriesbladerdeeg literblik abrikozen  eigensap  abrikozenjam sap     garnering vel...</td>
    </tr>
    <tr>
      <th>200</th>
      <td>bonen-groentesoep met rijst</td>
      <td>knolselderij  bleekselderij courgette  spekjes potten  bonen bouillontabletten kruidenbuiltje...</td>
    </tr>
    <tr>
      <th>201</th>
      <td>lamsstoofschotel "tunis"</td>
      <td>lamsschouder      runderbouillon artisjokken citroenen        gierst  doperwtjes  liefst      ...</td>
    </tr>
    <tr>
      <th>202</th>
      <td>spaghetti met garnalen in italiaanse roomsaus</td>
      <td>spaghetti   pasta  keuze  rauwe garnalen     roomboter      tuin   kerriepoeder     zeezout  s...</td>
    </tr>
    <tr>
      <th>203</th>
      <td>semur jawa</td>
      <td>mals rundvlees bieflappen  biefstuk  korianderpoeder   nootmuscaat     sjalotten    fijngeras...</td>
    </tr>
    <tr>
      <th>204</th>
      <td>maultaschen "gluckliches alter"</td>
      <td>varkensgehakt  kleingesneden chinese kool lenteuitjes  ringetjes geknipt  rijstwijn     gember...</td>
    </tr>
    <tr>
      <th>205</th>
      <td>oosterse wokschotel met tofu</td>
      <td>sofine tofu wokreepjes mild pikant    zonnebloemolie c selectie       fijngeraspte  gemberwort...</td>
    </tr>
    <tr>
      <th>206</th>
      <td>mosselsoep</td>
      <td>mosselen  preien   winterwortel  dille        margarine visbouillontablet</td>
    </tr>
    <tr>
      <th>207</th>
      <td>gerookte vis in kerrierijst</td>
      <td>droogkokende rijst         visbouillon    kerriepoeder  chilipoeder   makreel   dille</td>
    </tr>
    <tr>
      <th>208</th>
      <td>aardappelsalade met komkommer</td>
      <td>vleesbouillon   wijnazijn komkommer eierdooier              kervel</td>
    </tr>
    <tr>
      <th>209</th>
      <td>engels theebrood</td>
      <td>witmeel  brood    warme      gist</td>
    </tr>
    <tr>
      <th>210</th>
      <td>spek-dikken</td>
      <td>benodigdheden  roggemeel       keukenstroop      anijs    rookspek metwost   deed spek  rookwor...</td>
    </tr>
    <tr>
      <th>211</th>
      <td>fijne worstensneetjes</td>
      <td>knackebrod gistextract bv marmite  salami  leverworst augurkjes  chinese kool olievrije slasaus</td>
    </tr>
    <tr>
      <th>212</th>
      <td>gehaktbrood met gerookt ontbijtspek</td>
      <td>gemengd   runder     geperst          manis  paneermeel   ontbijtspek dun</td>
    </tr>
    <tr>
      <th>213</th>
      <td>ham</td>
      <td>aluminiumfolie   ham   kruidnagels</td>
    </tr>
    <tr>
      <th>214</th>
      <td>ratatouille</td>
      <td>aubergine courgette tomaat             rozemarijn  bonenkruid    laurierblaadje  cayennepeper</td>
    </tr>
    <tr>
      <th>215</th>
      <td>champignon-kabeljauw-balletjes</td>
      <td>zure   mierikswortelsaus  kabeljauwfilet ontveld    dichte       volkoren broodkruim   hazelno...</td>
    </tr>
    <tr>
      <th>216</th>
      <td>echte chocolademelk</td>
      <td></td>
    </tr>
    <tr>
      <th>217</th>
      <td>simpele snel-klaar pasta</td>
      <td>pasta fusseli  cashewnoten creme fraiche  kookroom garnalen kip roerbakgroentenmix  voeg    lek...</td>
    </tr>
    <tr>
      <th>218</th>
      <td>spruitjesbeignets</td>
      <td>spruitjes     ijswater</td>
    </tr>
    <tr>
      <th>219</th>
      <td>italiaans gekruide karbonades uit de oven</td>
      <td>schouderkarbonades    schaal     karbonades          oregano</td>
    </tr>
    <tr>
      <th>220</th>
      <td>rode poon met paddenstoelen en tomatensaus</td>
      <td>filets   poon  paddenstoelen         fijne            tomatensaus  gepeld ontpit    sjalot fijn...</td>
    </tr>
    <tr>
      <th>221</th>
      <td>goulash soep in de slow cooker</td>
      <td>mager rundvlees                       goede runderbouillon laurier</td>
    </tr>
    <tr>
      <th>222</th>
      <td>straaljager</td>
      <td>jenever frisdrank</td>
    </tr>
    <tr>
      <th>223</th>
      <td>tagliatelle met ham, prei en camembert</td>
      <td>ham   dik      ringen  camembert  tagliatelle     kookroom  kerriepoeder</td>
    </tr>
    <tr>
      <th>224</th>
      <td>radijsroosje</td>
      <td>radijsjes koud  ijsblokjes</td>
    </tr>
    <tr>
      <th>225</th>
      <td>zoetzure groenten</td>
      <td>winterwortel komkommer</td>
    </tr>
    <tr>
      <th>226</th>
      <td>bieflaprolletjes</td>
      <td>bieflappen  kalfslappen  doorregen spek  dun  sjalotten  zoetzure augurkjes     franse       s...</td>
    </tr>
    <tr>
      <th>227</th>
      <td>g e v u l d   e i t j e   a   l a   b r a b a n c o n n e</td>
      <td>zalm bld      sjalotjes   geklaarde        bieslookringetjes    garnituur</td>
    </tr>
    <tr>
      <th>228</th>
      <td>broccoli-zalmtaart</td>
      <td>bladerdeeg  broccoli         zalm              roomkaas sant moret  mon chou</td>
    </tr>
    <tr>
      <th>229</th>
      <td>mediterraanse mosselen met chilis&amp;comma zoete paprika en courgette</td>
      <td>mosselen   look    zoete  courgette   tajinne kruiden     sterk    hete chilis  keuze à  gnoc...</td>
    </tr>
    <tr>
      <th>230</th>
      <td>pompoensoep met sinaasappel&amp;comma kardemom&amp;comma kokosmelk en koriander</td>
      <td>pompoen   oranje hokkaido     gember     korianderpoeder kardemompeulen geplet     groentebou...</td>
    </tr>
    <tr>
      <th>231</th>
      <td>asperges met pasta, zalm en garnalen uit de oven</td>
      <td>asperges courgette   fusilli moot zalm   huid  roze gepelde garnalen  verstegen mix  scampi  ...</td>
    </tr>
    <tr>
      <th>232</th>
      <td>paastaart</td>
      <td>zelfrijzend bakmeel  bakpoeder       zachte   zonnebloemmargarine  schil    eiren opgeklopt wo...</td>
    </tr>
    <tr>
      <th>233</th>
      <td>bourdin met paprika en veel kruiden</td>
      <td>schouderlappen     varkenslever gespoeld  koud                selderij    cayennepeper        ...</td>
    </tr>
    <tr>
      <th>234</th>
      <td>humus (goemoes)</td>
      <td>kikkererwten  sesamolie     sap</td>
    </tr>
    <tr>
      <th>235</th>
      <td>groene aspergesalade</td>
      <td>asperges  pijnboompitten stevige  pomedore    tomatencoulis  gepelde              garnering  ...</td>
    </tr>
    <tr>
      <th>236</th>
      <td>jachtschotel</td>
      <td>gaar soepvlees  aardappelpuree        kruiden  smaak</td>
    </tr>
    <tr>
      <th>237</th>
      <td>sinaasappel cooler</td>
      <td>sinaasappelconcentraat   ontdooid bolletje sinaasappelsorbet ginger ale bolletje vanille ijs</td>
    </tr>
    <tr>
      <th>238</th>
      <td>limoncello blush</td>
      <td>limoncello citroenlikeur   schoongeboend  ijsblokjes  wodka  sprite frisdrank gekoeld  appel c...</td>
    </tr>
    <tr>
      <th>239</th>
      <td>griekse melk</td>
      <td>komkommer  korianderblaadjes    yoghurt</td>
    </tr>
    <tr>
      <th>240</th>
      <td>lamsvleessoep</td>
      <td>mager lamsvlees    fijngeraspte gemberwortel   schoongemaakte      vijfkruidenpoeder   laur...</td>
    </tr>
    <tr>
      <th>241</th>
      <td>cranberry-ijsthee[/b]</td>
      <td>thee vruchtensap</td>
    </tr>
    <tr>
      <th>242</th>
      <td>gamba`s in bagna càuda met watermeloen en rettich</td>
      <td>watermeloen   dun       gamba   garnalen  ansjovis        mild  roomboter knoflooktenen  truffe...</td>
    </tr>
    <tr>
      <th>243</th>
      <td>babyhapje: broccoli, kikkererwten en kokos</td>
      <td>kikkererwten  stronkje broccoli  kokos santen</td>
    </tr>
    <tr>
      <th>244</th>
      <td>macaronischotel met fougerond</td>
      <td>macaroni   bleekselderij       gtam  fougerond</td>
    </tr>
    <tr>
      <th>245</th>
      <td>gefrituurde sprotjes met mayonaise-dip.</td>
      <td>grofgehakt    knoflookzout  sprotjes plantaardige     frituren   kruidige mayonaisedip  mayo...</td>
    </tr>
    <tr>
      <th>246</th>
      <td>spareribs in de cola uit de slowcooker</td>
      <td>tagsspareribs slowcooker cola spareribs   slowcooker spareribs   cola</td>
    </tr>
    <tr>
      <th>247</th>
      <td>sambal goreng babi</td>
      <td>varkensvlees        knoflookteentje   gemberpoeder djahe trassi   sambal</td>
    </tr>
    <tr>
      <th>248</th>
      <td>chinees -roerei</td>
      <td>bosuitjes  tauge  ham  shitake paddestoelen  maizena  sherry     chilisaus</td>
    </tr>
    <tr>
      <th>249</th>
      <td>energie cocktail</td>
      <td></td>
    </tr>
    <tr>
      <th>250</th>
      <td>asperges</td>
      <td>asperges  zalm   aldi   sprieten</td>
    </tr>
    <tr>
      <th>251</th>
      <td>hollandse groentesoep met balletjes</td>
      <td>boerensoepgroenten koelvers  groentebouillon    rundersoepballetjes  wit casinobrood  jong...</td>
    </tr>
    <tr>
      <th>252</th>
      <td>peren/chocoladetaartjes</td>
      <td>rol   bladerdeeg rollen bladerdeeg  verkocht   koeling   supermarkt    bladerdeeg    laten ont...</td>
    </tr>
    <tr>
      <th>253</th>
      <td>paprika's in tomatensaus</td>
      <td>gezeefde       oregano   olijven    kruidenazijn</td>
    </tr>
    <tr>
      <th>254</th>
      <td>tapa van sperziebonen</td>
      <td>sperziebonen  afgehaald       ringen     kippebouillon tabletje     pers</td>
    </tr>
    <tr>
      <th>255</th>
      <td>kersensoep</td>
      <td>kopjes vlugkokende havermout kopjes     kersen</td>
    </tr>
    <tr>
      <th>256</th>
      <td>vakantie-rijst-schotel met kip</td>
      <td>rijst snelkook pandan potjes kippebouillon   vlees     poeder            smaak tauge</td>
    </tr>
    <tr>
      <th>257</th>
      <td>moscardini in guazzello allo zeuzero</td>
      <td>muskusinktvissen  snijbiet  spinazie   gepelde  driedubbel geconcentreerde   gember</td>
    </tr>
    <tr>
      <th>258</th>
      <td>sjieke toet</td>
      <td>goed roomijs  biologische halfvolle melktheelepel  kaneelscheutje licor</td>
    </tr>
    <tr>
      <th>259</th>
      <td>aardbeienbavarois</td>
      <td>aardbeien    tas   gelatine   poeder</td>
    </tr>
    <tr>
      <th>260</th>
      <td>tiramisu</td>
      <td>mascarpone   vingers  espresso  amaretto    cacaopoeder  poedersuiker</td>
    </tr>
    <tr>
      <th>261</th>
      <td>bbq - aardappelwaaiers met knoflookolie</td>
      <td>schoongeboend   geschild  knoflookolie</td>
    </tr>
    <tr>
      <th>262</th>
      <td>italiaanse gevulde champignons met gedroogde tomaatjes en kaas</td>
      <td>sjalotje        afhankelijk   grootte                 oregano  paneermeel los geklopt    b...</td>
    </tr>
    <tr>
      <th>263</th>
      <td>zaterdag voor bij de buis pan oranje soep</td>
      <td>sjalotjes groentebouillontabletten     rozemarijn  tomatensap  broodcroutons  tuinkruiden  sina...</td>
    </tr>
    <tr>
      <th>264</th>
      <td>gegratineerde rauwe andijviestamppot</td>
      <td>onderstaande hoeveelheden  richtlijnen varieer   smaak       voorgesneden andijvie   creme fra...</td>
    </tr>
    <tr>
      <th>265</th>
      <td>preiveloute.</td>
      <td>preien knolselder      selder aardappel   look  bouillon vetstof pezo kruidentuiltje  laurier  ...</td>
    </tr>
    <tr>
      <th>266</th>
      <td>pikante vruchtensalade</td>
      <td>annanas     perziken     gesnedenmiddelgrote goudrenetten    handperen     mandarijnen  mayo...</td>
    </tr>
    <tr>
      <th>267</th>
      <td>bonbonvulling</td>
      <td>nodig     kloppen      chocolade</td>
    </tr>
    <tr>
      <th>268</th>
      <td>roomsaus met gorgonzola en bleekselderij</td>
      <td>bleekselderij  walnoten  gorgonzola  roomsaus culinair</td>
    </tr>
    <tr>
      <th>269</th>
      <td>knolselderijsoep met een overheerlijke pesto van rucola en walnoten</td>
      <td>knolselderij   appels      groentebouillon     pesto  walnoten  rucola    parmezaanse  grof ger...</td>
    </tr>
    <tr>
      <th>270</th>
      <td>gevulde mexicaanse slavinken</td>
      <td>tacokruiden       sjalotje        belegen leidse      ontbijtspek  gerold    maggi vloeib...</td>
    </tr>
    <tr>
      <th>271</th>
      <td>thais stoof­pot­je</td>
      <td>rib runderlappen     currypasta   sesamolie   zonnebloemolie  rundvleesbouillon    pandanrijst...</td>
    </tr>
    <tr>
      <th>272</th>
      <td>panga(of forel)filet met chiliboter, kruidenaardappelpuree</td>
      <td>forel  zalmfilets   per       vis   chili knoflookboter</td>
    </tr>
    <tr>
      <th>273</th>
      <td>soep</td>
      <td>kip tomaat kruiden   bereidingswijze</td>
    </tr>
    <tr>
      <th>274</th>
      <td>kip in het groene paradijs</td>
      <td>kippenbouillon   broccoli  roosjes    shii take  oestersaus  lichte   chinese sesamolie  s...</td>
    </tr>
    <tr>
      <th>275</th>
      <td>sable aux fraises-aardbeiensables</td>
      <td>deeg      poedersuiker gezeefd    eierdooiers   druppel vanille  citroenessence  aardbeien   ...</td>
    </tr>
    <tr>
      <th>276</th>
      <td>stoverij van melkzwammen met paprika en knoflook</td>
      <td>melkzwammen     gele              molen</td>
    </tr>
    <tr>
      <th>277</th>
      <td>"pepernoten" met amaretto en abrikozen</td>
      <td>bakpoeder  fijne    piment  gemberpoeder  kardamom     kruidnagell    amaretto  magere    ...</td>
    </tr>
    <tr>
      <th>278</th>
      <td>tiramisu</td>
      <td>eidooiers      ongezoet koffie sterk  cafe marakesh likeur  amaretto   vingers cacou poeder mas...</td>
    </tr>
    <tr>
      <th>279</th>
      <td>kalfssucade met salie en witte wijn</td>
      <td>kalfssucadelappen trostomaten      sjalotten     kaneelpoeder        salie</td>
    </tr>
    <tr>
      <th>280</th>
      <td>glace d'amandes et agrumes pochee</td>
      <td>gebrande amandelschaafsel eierdooiers        mascarpone amaretto receptuur zuidvruchten  zuidv...</td>
    </tr>
    <tr>
      <th>281</th>
      <td>uiensoep</td>
      <td>bastard     runderbouillon      stokbrood   geperst   cheddar  camembert</td>
    </tr>
    <tr>
      <th>282</th>
      <td>chocolademousse met karamelsaus</td>
      <td>beker chocolademousses  pistachenoten</td>
    </tr>
    <tr>
      <th>283</th>
      <td>snelle soep met thaise roerbakgroente en kip - koolhydraat beperkt</td>
      <td>runderbouillonblokjes groentebouillonblokjes thaise oosterse roerbakgroenten kip</td>
    </tr>
    <tr>
      <th>284</th>
      <td>kalkoensalade met geroosterde hazelnoten</td>
      <td>avocado kerstomaatjes  hazelnoot  kalkoenfilet  frisee  sinaasappelsap    dragonazijn  cajunkru...</td>
    </tr>
    <tr>
      <th>285</th>
      <td>cajun kip gevuld met kaas en spinazie</td>
      <td>cheddar  münster geraspt  diepvriesspinazie ontdooid  uitgelekt  spinazie    plantaardige   ...</td>
    </tr>
    <tr>
      <th>286</th>
      <td>broccolli-salade</td>
      <td>broccoli  blanke rozijnen  feta  zonnebloempitten    yofresh slasaus</td>
    </tr>
    <tr>
      <th>287</th>
      <td>kerrie spruitjes</td>
      <td>spruitjes     kerriepoeder    santen conimex</td>
    </tr>
    <tr>
      <th>288</th>
      <td>tongkrullen op een spinaziesalade met garnalenmayonaise</td>
      <td>ontvelde tongfilets  hollandse garnalen  mayonaise  zure   frambozen     walnotenolie  jonge sp...</td>
    </tr>
    <tr>
      <th>289</th>
      <td>schrobbeler</td>
      <td>likeur</td>
    </tr>
    <tr>
      <th>290</th>
      <td>pilaf met rundvlees</td>
      <td>perziken blikken gepelde       sterke bouillon  rundvlees poulet   rozijnen  piment  kurkuma k...</td>
    </tr>
    <tr>
      <th>291</th>
      <td>mosselsaus</td>
      <td>mayonaise         molen mosselnat gezeefd</td>
    </tr>
    <tr>
      <th>292</th>
      <td>groentensamosa</td>
      <td>deeg  gezeefde     bakpoeder    vulling    gesnipperde   wortelen  geschilde        erwten  s...</td>
    </tr>
    <tr>
      <th>293</th>
      <td>gebakken zoete aardappelen</td>
      <td>zoete    ongezouten roomboter          kardamon  whiskey</td>
    </tr>
    <tr>
      <th>294</th>
      <td>penne con pesto alla calabrese</td>
      <td>penne     ricotta     salsa di pomodoro     pecorino romano  grana padano spaanse petertjes or...</td>
    </tr>
    <tr>
      <th>295</th>
      <td>spaghettisaus met veel groenten</td>
      <td>heinz pikante saus  spaghetti winterwortel     ringen courgette   rundergehakt doos kersentoma...</td>
    </tr>
    <tr>
      <th>296</th>
      <td>kwark met ananas</td>
      <td>kwark    fruit bv ananas mandarijntjes  d</td>
    </tr>
    <tr>
      <th>297</th>
      <td>pikante dadels (tapas)</td>
      <td>geconfijte dadels lapjes ontbijtspek  bacon houten cocktailprikkers</td>
    </tr>
    <tr>
      <th>298</th>
      <td>simpele rijst met prut</td>
      <td>varkenspoulet   ananas        tomatenketchup    rijst</td>
    </tr>
    <tr>
      <th>299</th>
      <td>kruidige eieren</td>
      <td>dragon  dille   sjalotjes  dieet margarine     dragonazijn  kappertjes snee bruinbrood</td>
    </tr>
    <tr>
      <th>300</th>
      <td>crokant,romig hoorntje.</td>
      <td>hoorntjes    poedersuiker    zoete sherry  sterke  koffie frituurolie vormpjes   hoorntjes  ma...</td>
    </tr>
    <tr>
      <th>301</th>
      <td>kiprolletjes met graanmosterd en dragon.</td>
      <td>kipfilets  gerookt spek    tomaatjes   tomaatjes  paneermeel  bearnaisesaus  graanmosterd  prov...</td>
    </tr>
    <tr>
      <th>302</th>
      <td>mini-cake</td>
      <td>margarine  pure chocolade    eidooiers          bestuiven</td>
    </tr>
    <tr>
      <th>303</th>
      <td>zuppa di cozzo alla marinara</td>
      <td>mosselen dcl    grof    grof  bleekselderijstenges  smalle boogjes  citroenschil  centimeter ...</td>
    </tr>
    <tr>
      <th>304</th>
      <td>zalmpakketjes met tijmboter</td>
      <td>moten zalm    zachte    geplet  mosterdzaadjes   tijmnaaldjes</td>
    </tr>
    <tr>
      <th>305</th>
      <td>kruidige kipspiesjes met fruitige kerriesaus</td>
      <td>traditioneel c selectie rasp      schijf ananas   mandarijn  fijngeknipt   calve kerrie a...</td>
    </tr>
    <tr>
      <th>306</th>
      <td>pittige kerrie</td>
      <td>rijst  indiase kerriepasta patak       sap bouillon poeder     bezien bouillonpoeder ajuinen k...</td>
    </tr>
    <tr>
      <th>307</th>
      <td>citroendressing.</td>
      <td>mayonaise</td>
    </tr>
    <tr>
      <th>308</th>
      <td>sinterklaasbrood</td>
      <td>tarwemeel  lauwe    gist     kardamon  gesmolten     rozijnen  amandelen  versiering</td>
    </tr>
    <tr>
      <th>309</th>
      <td>gegrilde, malse thaise koriander kip</td>
      <td>vissaus nam pla  sesamolie    chilipoeder</td>
    </tr>
    <tr>
      <th>310</th>
      <td>bbq chicken pizza</td>
      <td>turks brood kant  klare pizza bodem  zelfgemaakte bodem   saus   bodem                         ...</td>
    </tr>
    <tr>
      <th>311</th>
      <td>ansjovis-knoflooksaus</td>
      <td>geperst  ansjovisfilets    uitgelekt  grofgehakt  heet</td>
    </tr>
    <tr>
      <th>312</th>
      <td>cicche del nonno (opa's peuken)</td>
      <td>aardappels for boiling but not new aardappels coarse grained  lbs spinazie large stems removed...</td>
    </tr>
    <tr>
      <th>313</th>
      <td>sherry &amp; porto flip</td>
      <td>port</td>
    </tr>
    <tr>
      <th>314</th>
      <td>linzentaart</td>
      <td>linzen       gemengde  kruiden  gemberpoeder</td>
    </tr>
    <tr>
      <th>315</th>
      <td>kruidenmix voor ovengebakken kip</td>
      <td>kippenbouten benodigd   kippenbouten kruidenmix       grof zeezout    korianderpoeder  knofloo...</td>
    </tr>
    <tr>
      <th>316</th>
      <td>stoofpeertjes</td>
      <td>gieser wildeman peertjes fles  bordeaux    kruidnagelen</td>
    </tr>
    <tr>
      <th>317</th>
      <td>banana</td>
      <td>vruchtensap</td>
    </tr>
    <tr>
      <th>318</th>
      <td>italiaanse zeevruchtensalade</td>
      <td>pijlinktvis  mossel  venusschelp  gamba  bleekselderij wortels        peperkorrels laurierblad...</td>
    </tr>
    <tr>
      <th>319</th>
      <td>stoofpot van wortelen</td>
      <td>kippenbouillon laurierblaadjes peer  worsten varken kalf  worteltjes sjalotje snuifjes...</td>
    </tr>
    <tr>
      <th>320</th>
      <td>aardappeltaart</td>
      <td>vastkokende   gezouten     broodkruimels   type oude brugse   type passendale</td>
    </tr>
    <tr>
      <th>321</th>
      <td>ovenschotel met zalm en verse spinazie</td>
      <td>spinazie   zalm     euroshopper  ah gekocht heerlijk  pittige            culinair  culinair l...</td>
    </tr>
    <tr>
      <th>322</th>
      <td>lamsfilet in korstdeeg</td>
      <td>lamsfilet   bladerdeeg      ham bussel    dragon sjalotjes uitje          kalfsfond  notenolie</td>
    </tr>
    <tr>
      <th>323</th>
      <td>ketjapkabonades</td>
      <td>varkenskarbonade    sambal oelek sap</td>
    </tr>
    <tr>
      <th>324</th>
      <td>tomatensalade met gorgonzola</td>
      <td>vleestomaten    gorgonzola  zure</td>
    </tr>
    <tr>
      <th>325</th>
      <td>bont slaatje met scampi</td>
      <td>scampis sap   verschillende soorten sla slamix avocado sinaasappel      dressing sap  sinaasap...</td>
    </tr>
    <tr>
      <th>326</th>
      <td>johan hofman nachoschotel</td>
      <td>naturel nacho chips  burritomix milde salsasaus mais    groente</td>
    </tr>
    <tr>
      <th>327</th>
      <td>pylsur</td>
      <td>zachte  puntjes chipolata worstjes pylsussinep zoete  saus   zaad   blond bier pils      knoflo...</td>
    </tr>
    <tr>
      <th>328</th>
      <td>frambozenpudding met druiven.</td>
      <td>frambozenpuddingpoeder kookpudding   creme  cassis   druiven   frambozen  kirsch</td>
    </tr>
    <tr>
      <th>329</th>
      <td>bollen met krieken.</td>
      <td>platte    vingers           krieken   hazelnoten  bloemsuiker</td>
    </tr>
    <tr>
      <th>330</th>
      <td>zalige ovenschotel met doperwtjes en zalm</td>
      <td>balsamico         zonnebloemolie  doperwten uitlekgewicht   doperwten     lekkerder   zalm   z...</td>
    </tr>
    <tr>
      <th>331</th>
      <td>warme uien met haringsalade en gebakken aardappelen</td>
      <td>laurierbladeren peperkorrels   smaak  mosterdzaad haringfilets maatjes  creme fraiche   ap...</td>
    </tr>
    <tr>
      <th>332</th>
      <td>knoflooksaus</td>
      <td>mayonaise  yoghurt</td>
    </tr>
    <tr>
      <th>333</th>
      <td>bietjes-tonijn-aardappelsalade</td>
      <td>gekookte  bietjes  tonijn   gekookt  zilveruitjes augurken  ananas   mandarijnen appel gekookte...</td>
    </tr>
    <tr>
      <th>334</th>
      <td>witlofstampot met appel&amp;commakruiden&amp;comma geraspte kaas en ketjap</td>
      <td>bepalen      neemt voldoenden witlof stampotaardappelen geschilde appel      jongen soja   ...</td>
    </tr>
    <tr>
      <th>335</th>
      <td>spicy wrapfest</td>
      <td>wraps    maken   feestgangers staan  wachten  nee  biologische kip   thuis hangen  slingers  g...</td>
    </tr>
    <tr>
      <th>336</th>
      <td>kerriesaus en kaas-roomsaus</td>
      <td>kerriesaus  margarine     kerriepoeder  volkorenmeel  kruidenbouillon    koffieroomzoutvoor  ...</td>
    </tr>
    <tr>
      <th>337</th>
      <td>sinaasappelsoep/soupe a l`orange</td>
      <td>sinaasappels    heet   sinaasappellikeur  cointreau  ijs vanille roomijs  zelfgemaakt sinaasap...</td>
    </tr>
    <tr>
      <th>338</th>
      <td>ovenschotel met siciliaanse wostjes</td>
      <td>siciliaanse worstjes  soort  natuurlijk  schoongemaakte  worteltjes   grove    grove   schoonge...</td>
    </tr>
    <tr>
      <th>339</th>
      <td>flensjes gevuld met advocaatcreme</td>
      <td>advocaat      vanillevla</td>
    </tr>
    <tr>
      <th>340</th>
      <td>carlito</td>
      <td>wodka  frisdrank</td>
    </tr>
    <tr>
      <th>341</th>
      <td>zoet zure spareribs</td>
      <td>spareribs zoet zure chilisaus toko bloemen</td>
    </tr>
    <tr>
      <th>342</th>
      <td>frittata</td>
      <td>bourgondische krieltjes   middel        tuinkers</td>
    </tr>
    <tr>
      <th>343</th>
      <td>gepofte appel uit de oven</td>
      <td>goud reinette kaneelstokjes  basterdsuiker  roomboter kruidnagel  rozijnen</td>
    </tr>
    <tr>
      <th>344</th>
      <td>marokkaans kipstoofpotje</td>
      <td>gemberpoeder   gember geraspt     gember  laat     komijnpoeder       smaak  ras  hanout verkr...</td>
    </tr>
    <tr>
      <th>345</th>
      <td>thaise pad thai met tijgergarnalen</td>
      <td>tijgergarnalen  tofu    garnalen verkrijgbaar   toko     taugé  bouillon   pinda   meiasia ing...</td>
    </tr>
    <tr>
      <th>346</th>
      <td>bloemkoolsalade</td>
      <td>bloemkool    roosjes     druiven gehalveerd   rozijnen appel     rijpe banaan à mandarijnen  pa...</td>
    </tr>
    <tr>
      <th>347</th>
      <td>kleine mozarella bolletjes met munt en chilipeper</td>
      <td>bocconcini mozarella  bolletjes mozarella   neemt   mozarella      chillivlokken    pepertje  ...</td>
    </tr>
    <tr>
      <th>348</th>
      <td>'pastekop' van de zee</td>
      <td>ongepelde grijze noordzeegarnalen  kreukels  wulken  gelatine   vocht   bouillon wortel   seld...</td>
    </tr>
    <tr>
      <th>349</th>
      <td>mexicaanse tomaten soup</td>
      <td>blikes gepelde tomaat     ah      recept werkt        sjalotten      smaak</td>
    </tr>
    <tr>
      <th>350</th>
      <td>traktatie banaan met chocolade</td>
      <td>bananen  pure chocolade gekleurde spikkels stokjes  cakepop stokjes  lolly stokjes</td>
    </tr>
    <tr>
      <th>351</th>
      <td>heerlijk fris romige citroentaart in zonnebloem vorm</td>
      <td>diepvriesbladerdeeg citroenen   roomkaas   poedersuiker     zoetekauwen  custardpoeder</td>
    </tr>
    <tr>
      <th>352</th>
      <td>kippenbouillon</td>
      <td>kip      schoongemaakte    gesnedsen  schoongemaakte         worteltjes   geplette  peperkorrel...</td>
    </tr>
    <tr>
      <th>353</th>
      <td>zoete marokkaanse broodjes</td>
      <td>lauwe   zachte   vanillesuiker        gist kommetje abrikozenjam kommetje gesmolten     kokos</td>
    </tr>
    <tr>
      <th>354</th>
      <td>aardappel-groentepannetje met spek en kaas</td>
      <td>lente uitjes  mon chou  zeeuws spek    voorgekookte aardappelblokjes  diepvriesdoperwten</td>
    </tr>
    <tr>
      <th>355</th>
      <td>mosselsoep</td>
      <td>overschot   mosselen     pers  kookvocht   groenten   gekookte mosselen        selder sommigen...</td>
    </tr>
    <tr>
      <th>356</th>
      <td>hele zalm op de barbecue&amp;comma met een kruidensausje</td>
      <td>pacific zilverzalm   soepgroenten          gedroogd  laurier      saus porties kervel    sherry...</td>
    </tr>
    <tr>
      <th>357</th>
      <td>pizza met aubergine, knoflookworst en gamba's</td>
      <td>deeg   pizza meel     saus      puree    italiaanse kruiden pizzabeleg aubergine gele   knofloo...</td>
    </tr>
    <tr>
      <th>358</th>
      <td>sienacake.</td>
      <td>geroosterde geblancheerde amandelen  geroosterde hazelnoten  gekonfijte abrikozen   gekonfijte...</td>
    </tr>
    <tr>
      <th>359</th>
      <td>aardappelsalade met knakworst, tomaat en parmezaanse kaas</td>
      <td>gaar geschild    tomaat  zaden   knakworstjes   parmezaanse  grof geraspt mayonaise slabladeren</td>
    </tr>
    <tr>
      <th>360</th>
      <td>vlees gemarineerd in augurkennat</td>
      <td>augurkennat  benodigd    cayennepeper   smaak   meng  marinade ingrediënten  marineer  vlees  ...</td>
    </tr>
    <tr>
      <th>361</th>
      <td>gevulde sardinefilets</td>
      <td>bosuitje zongedroogde      sprietjes   gemarineerde sardinefilets  ah</td>
    </tr>
    <tr>
      <th>362</th>
      <td>warme vijgen met gorgonzola</td>
      <td>gorgonzola   vijgen</td>
    </tr>
    <tr>
      <th>363</th>
      <td>couscous met gewokte pompoen &amp; knolselder &amp; spinazie</td>
      <td>couscous  pompoen     knolselder      spinazie       nodig  gembersiroop geitenkaas afwerking ...</td>
    </tr>
    <tr>
      <th>364</th>
      <td>vulling voor ?braad-vogels?</td>
      <td>oudbakken harde bolletjes  stokbrood    magere rookspek  kippenlevertjes doperwten       roombo...</td>
    </tr>
    <tr>
      <th>365</th>
      <td>gazpacho</td>
      <td>asperges    kippenbouillon    zaadjes gele   zaadjes</td>
    </tr>
    <tr>
      <th>366</th>
      <td>roodbaars in papillotte</td>
      <td>roodbaarsfilets    tomatenpassata limoenen  olijven      geplette</td>
    </tr>
    <tr>
      <th>367</th>
      <td>ovenvers frans broodje met kruidenroomkaas</td>
      <td>roomkaas  gestapte    creme fraiche     geknipte  bake off pistolets gemengde sla</td>
    </tr>
    <tr>
      <th>368</th>
      <td>verrukkelijke kruimel-cupcakes met peer en blauwe bessen</td>
      <td>goed rijpe peer conference  perfect geschild       blauwe bessen     vries  poedersuiker    ...</td>
    </tr>
    <tr>
      <th>369</th>
      <td>pompoenbrood</td>
      <td>poepoenvlees  gesmolten    volle  bakpoeder  tweederde  meel   ½theelepel        walnoten</td>
    </tr>
    <tr>
      <th>370</th>
      <td>gemakkelijke mascarponecreme met bitterkoekjes</td>
      <td>amaretti amandelbitterkoekjes  amaretto  mascarpone  fijne   poedersuiker  pure chocolade</td>
    </tr>
    <tr>
      <th>371</th>
      <td>nestjes met koolvis</td>
      <td>nestjes         eierdooier  eiwit   vulling  koolvisplakjes    diepvriesdoperwten    tabasco ...</td>
    </tr>
    <tr>
      <th>372</th>
      <td>gegrilde kip met lenteui, pittig</td>
      <td>hete pepersaus     lenteuitjes kipfilets</td>
    </tr>
    <tr>
      <th>373</th>
      <td>pizzoccheri</td>
      <td>personen   nodig  pizzoccheri       savooiekool   stevige aardappels    taleggio  fontina   ku...</td>
    </tr>
    <tr>
      <th>374</th>
      <td>witte bonen huts</td>
      <td>personen royaal puree i    bonen   saus   bonen goed laten uitlekken i   rookworst       chili...</td>
    </tr>
    <tr>
      <th>375</th>
      <td>toemis champignons&amp;period &amp;lpargestoofde&amp;comma gekruide champignons&amp;rpar</td>
      <td>salamblaadjes     asemwater     smaak  lombok   smaak serehstengel</td>
    </tr>
    <tr>
      <th>376</th>
      <td>tomaten-aardappelenpuree</td>
      <td>smaak         cayennepeper</td>
    </tr>
    <tr>
      <th>377</th>
      <td>varkensschnitzel met brie</td>
      <td>varkensschnitzels ongepaneerd    brie      lenteuitjes</td>
    </tr>
    <tr>
      <th>378</th>
      <td>hazelnoottaartjes met slagroomglazuur.</td>
      <td>tarwebloem      uitrollen    koude roomboter  magarine   ectra   bakvormpjes eidooiers  blanke...</td>
    </tr>
    <tr>
      <th>379</th>
      <td>kip basquaise in 't pannetje uit frans baskenland</td>
      <td>piment d espelette  vleugje chili  zoet  biologische kip à         ringen           gele   ...</td>
    </tr>
    <tr>
      <th>380</th>
      <td>pastadeeg eenvoudige bereiding</td>
      <td></td>
    </tr>
    <tr>
      <th>381</th>
      <td>konijn met tripelbier</td>
      <td>konijn flesjes tripel    wittewijnazijn tros druiven maizena express</td>
    </tr>
    <tr>
      <th>382</th>
      <td>linguine met rucola en kaas&amp;period</td>
      <td>linguine  smeerbare   philadelphia  rucola   vergine    parmezaanse  preien      behoefte</td>
    </tr>
    <tr>
      <th>383</th>
      <td>hollandse spitskool</td>
      <td>classico bertolli  ontbijtspek     ringen     spitskool</td>
    </tr>
    <tr>
      <th>384</th>
      <td>groente-noten-beignets</td>
      <td>amandelen   courgette   wortel   pompoen                   bakken</td>
    </tr>
    <tr>
      <th>385</th>
      <td>visfondue met heerlijke sausjes</td>
      <td>visbouillon  opgelost   heet  wortel sjalot        laurier   warme uiensausje          radijss...</td>
    </tr>
    <tr>
      <th>386</th>
      <td>peches a la napoleon</td>
      <td>rijpe perziken  cognacbitterkoekjesstijfgeslagen</td>
    </tr>
    <tr>
      <th>387</th>
      <td>lekkerbekjes van pastoor</td>
      <td>lekkerbekjes     kokos limoen appels bosuitjes  gemberjam  sambal oelek winterwortel</td>
    </tr>
    <tr>
      <th>388</th>
      <td>cranberry bread</td>
      <td>walnoten  gesmolten      sinaasappelsap kopjes   bakpoeder   sinaasappelschil      bevroren cr...</td>
    </tr>
    <tr>
      <th>389</th>
      <td>eyecatchers voor soepjes&amp;period</td>
      <td>bereiding</td>
    </tr>
    <tr>
      <th>390</th>
      <td>lauwwarme aardappel-bietensalade</td>
      <td>vastkokende  geschild      oo  gekookte bieten geschild       hamblokjes  zoetzure augurkjes  ...</td>
    </tr>
    <tr>
      <th>391</th>
      <td>tomaten-groentesoep</td>
      <td>tomatenblokjes  sap   maggi basis  tomatencremesoep  fijne soepgroente laurierblaadje  maggi s...</td>
    </tr>
    <tr>
      <th>392</th>
      <td>creme karamel</td>
      <td>karamel     custard  warme      vanille essence</td>
    </tr>
    <tr>
      <th>393</th>
      <td>portugese biscuitgebak</td>
      <td>gesplitst      druppeltjes vanille  amandelessence</td>
    </tr>
    <tr>
      <th>394</th>
      <td>wasabi zalm &amp; wilde rijst</td>
      <td>wasabipasta  arachideolie  zalmfilet  vel     rijst lente uitjes   zoute   handenvol rucola</td>
    </tr>
    <tr>
      <th>395</th>
      <td>cachaca collins</td>
      <td>vruchtensap</td>
    </tr>
    <tr>
      <th>396</th>
      <td>geglaceerde ham met cajunkruiden</td>
      <td>achterham  mm dik  sjalotje  bleekselderij  winterpeen    cajunkruiden  abrikozenjam  heldere ...</td>
    </tr>
    <tr>
      <th>397</th>
      <td>kool met cashewnoten</td>
      <td>kool       roerbakken   cashewnoten   zoute   chilipoeder    appel  ananas gebakken</td>
    </tr>
    <tr>
      <th>398</th>
      <td>vrijgezel</td>
      <td>cognac</td>
    </tr>
    <tr>
      <th>399</th>
      <td>uitgebeend kippenrol met champignons</td>
      <td>uitgebeende kip  vel              oregano         salie</td>
    </tr>
    <tr>
      <th>400</th>
      <td>casselerrib met knackworst, rookspek en witte kool</td>
      <td>casselerrib knackworsten     mager gerookt spek snij      spek  hamschijf  varkenspoot   kool ...</td>
    </tr>
    <tr>
      <th>401</th>
      <td>texaans gehaktmengsel</td>
      <td>blikken texaansgroente mengsel  tortillachips  rundergehakt  zure    oude</td>
    </tr>
    <tr>
      <th>402</th>
      <td>pittige drumsticks</td>
      <td>drumsticks thaise red curry paste</td>
    </tr>
    <tr>
      <th>403</th>
      <td>kip pulau</td>
      <td>kip  lever    manis  zoete chilisaus  zonnebloemolie  arachideolie knoflookteentjes geperst  r...</td>
    </tr>
    <tr>
      <th>404</th>
      <td>pasta met verse tonijn</td>
      <td>kappertjes ontpitte  olijven   tonijnfilet  holle pasta</td>
    </tr>
    <tr>
      <th>405</th>
      <td>koninginnekoek</td>
      <td>vormen   middellijn     basterdsuiker  roomboter   citroenrasp</td>
    </tr>
    <tr>
      <th>406</th>
      <td>quiche met artisjokken</td>
      <td>rol bladerdeeg    blauwe  blue d auvergne roquefort stilton      uitgelekte artisjokharten     ...</td>
    </tr>
    <tr>
      <th>407</th>
      <td>broodje met avocado en ei&amp;period</td>
      <td>stok brood avocado gekookte    zeezout tuinkers</td>
    </tr>
    <tr>
      <th>408</th>
      <td>kipsalade met bamboespruiten en palmhartjes</td>
      <td>gaargekookt  braadkuiken palmhartjes bamboespruiten  ananas  olijven saus hardgekookte  rauwe e...</td>
    </tr>
    <tr>
      <th>409</th>
      <td>fusion biefstuk</td>
      <td>biefstuk  mon chou pan oven bosuitje  japanse      kerrie</td>
    </tr>
    <tr>
      <th>410</th>
      <td>aspergetaart.</td>
      <td>bladerdeeg paneermeel  asperges uitgelekt       hardgekookte  grofgehakt  creme fraiche</td>
    </tr>
    <tr>
      <th>411</th>
      <td>focaccia met tomaat en olijven</td>
      <td>tarwemeel  gist     oregano gedroogd   gedroogd  kruiden  brushetta     zongedroogde      olij...</td>
    </tr>
    <tr>
      <th>412</th>
      <td>thaise basilicum steak</td>
      <td>rib eye steak  rosbief aubergine   meiasia wokolie  meiasia thaise  steak woksaus</td>
    </tr>
    <tr>
      <th>413</th>
      <td>gevulde kersttomaatjes met ham en doperwten.</td>
      <td>kersttomaatjes   doperwten gham      tuinkers  mayonaise</td>
    </tr>
    <tr>
      <th>414</th>
      <td>polderkoek</td>
      <td>tarwebloem    koffie       gerookt  gedroogd vet spek</td>
    </tr>
    <tr>
      <th>415</th>
      <td>sambal goreng telor</td>
      <td>sambal oelek trassi    laos  gesnipperde   daun salam laurier      santenmix  sojaolie</td>
    </tr>
    <tr>
      <th>416</th>
      <td>bobotie</td>
      <td>snee brood    kerrie         laurierbladeren</td>
    </tr>
    <tr>
      <th>417</th>
      <td>snert</td>
      <td>spliterten knolserderij ong dobbelsteentjes    soepgroenten iglo  maggie  schouderkarbonades r...</td>
    </tr>
    <tr>
      <th>418</th>
      <td>ijssouffle van champagne en grappa</td>
      <td>eierdooiers      champagne  grappa  plaats  grappa   marc  champagne  genomen</td>
    </tr>
    <tr>
      <th>419</th>
      <td>patersgebraad.</td>
      <td>leenstuk uitgebeend varkensvlees      bier  hoge gisting abdijbier thijm rozemarijn muskaat   ...</td>
    </tr>
    <tr>
      <th>420</th>
      <td>pompoenbrood</td>
      <td>pompoenmoes       gist  lauw</td>
    </tr>
    <tr>
      <th>421</th>
      <td>tam konijn</td>
      <td>tam konijn  delen    margarine     konijnkruiden</td>
    </tr>
    <tr>
      <th>422</th>
      <td>tonijnfilet van de grill of de barbecue</td>
      <td>tonijnfilet   provençaalse ruiden</td>
    </tr>
    <tr>
      <th>423</th>
      <td>barbecue saus met pit</td>
      <td>barbecuesaus  bloemenhoning  sinaasappelmarinade   abrikozen uitgelekt      eventuel  chilipoe...</td>
    </tr>
    <tr>
      <th>424</th>
      <td>authentiek zuid-indiaas "dal fry" recept &amp;lpardahl&amp;comma dhal&amp;rpar</td>
      <td>linzen     kurkuma geelwortelpoeder   mosterzaadjes  chili  gebroken  kerrie  curry leaves  n...</td>
    </tr>
    <tr>
      <th>425</th>
      <td>pizzatosti</td>
      <td>boterhammen   jong belegen    salami tomatenketchup  italiaanse keukenkruiden</td>
    </tr>
    <tr>
      <th>426</th>
      <td>&amp;astgemarineerde karbonades met tomaten en sperziebonensalade&amp;ast</td>
      <td>lente bosuitjes       zaadjes     tijmblaadjes   gedroogd  honin  schouderkarbonades    sperzie...</td>
    </tr>
    <tr>
      <th>427</th>
      <td>fijne pate</td>
      <td>werk        runderstaartstuk        piment  ongezouten kalfsbouillon  poedergelatinetakjes ...</td>
    </tr>
    <tr>
      <th>428</th>
      <td>gemengde sla met notenkaas en peer</td>
      <td>peren    fromage aux noix ah  walnoten   zonnebloemolie   wijnazijn  vloeibare       gemengde ...</td>
    </tr>
    <tr>
      <th>429</th>
      <td>warm appeltaartje met calvados</td>
      <td>volkoren ontbijtkoek     appels  custardpoeder  rozijnen  calvados  appelstroop  vanillevla</td>
    </tr>
    <tr>
      <th>430</th>
      <td>flammkuchen met groene asperges&amp;comma champignons&amp;comma coburgerham en gruyère</td>
      <td>crème fraiche     aspergetips  kastanjechampignons  coburgerham   gruyère</td>
    </tr>
    <tr>
      <th>431</th>
      <td>gemarineerd lamskoteletten op bbq</td>
      <td>lamskoteletten   rib  marinade  saus  zonnebloemolie    ringen     laurierbladeren gescheurd  ...</td>
    </tr>
    <tr>
      <th>432</th>
      <td>sajoer boontjes met valess filetstukjes</td>
      <td>sperziebonen      fijngeraspte  laos gember   lekker ipv laos  santen  kokosmelk  basterdsuike...</td>
    </tr>
    <tr>
      <th>433</th>
      <td>pesto-eieren</td>
      <td>hardgekookte       parmezaanse</td>
    </tr>
    <tr>
      <th>434</th>
      <td>pepernoten</td>
      <td>mooie schaal   zelfrijzend bakmeel  donkerbruine basterdsuiker    margarine    speculaaskruid...</td>
    </tr>
    <tr>
      <th>435</th>
      <td>gesmoorde kip met paprika</td>
      <td>gele     kippenbouillon    vergine</td>
    </tr>
    <tr>
      <th>436</th>
      <td>tapas van gefrituurde ansjovis-boquerones fritos</td>
      <td>ansjovisjes         partjes</td>
    </tr>
    <tr>
      <th>437</th>
      <td>vijgen met serranoham</td>
      <td>vijgen  serranoham sherry  basilicumblaadjes grove</td>
    </tr>
    <tr>
      <th>438</th>
      <td>tjap tjoi op mijn manier</td>
      <td>kipfilets  hamlappen     sherry  bleekselderij  tauge  worteltjes garm chinese kool bosuitjes ...</td>
    </tr>
    <tr>
      <th>439</th>
      <td>krokante kip met kokos</td>
      <td>kokosrasp  kruiden  keuze    lekker vindt smaken  kip  kokos kokosolie puur   ontgeurd</td>
    </tr>
    <tr>
      <th>440</th>
      <td>fijne boerenkoolstamppot met knoflook</td>
      <td>boerenkool  kruimig kokende           molen</td>
    </tr>
    <tr>
      <th>441</th>
      <td>basissoep</td>
      <td>preien     bouillonpasta   thym laurier   margarine</td>
    </tr>
    <tr>
      <th>442</th>
      <td>tropicana kwark</td>
      <td>bereidingstijd ananasschijven   volle citroenkwark   creme fraiche   cruesli multi fruit zeef ...</td>
    </tr>
    <tr>
      <th>443</th>
      <td>mihoen in tomatensaus met sajoer tjampoer</td>
      <td>mihoen  mihoen          knijper  maisolie     varkensfricandeau    maïzena</td>
    </tr>
    <tr>
      <th>444</th>
      <td>lime pie</td>
      <td>marie koekjes  digestives à  roomboter  gecondenseerde  sap  à limoenen    schil  limoenen eit...</td>
    </tr>
    <tr>
      <th>445</th>
      <td>gestoofde braadlappen met tomaat, boontjes en tagliatelle</td>
      <td>runder braadlappen   pommodori           tagliatelle  haricot verts</td>
    </tr>
    <tr>
      <th>446</th>
      <td>spaghetti met vis</td>
      <td>kabeljauwfilet     gekookte mosselen     spaanse      laurierblaadjes      gepelde     spaghetti</td>
    </tr>
    <tr>
      <th>447</th>
      <td>kip in shasliksaus</td>
      <td>sjalotjes    shasliksaus calve  rijst lekkerste  pandan rijst kipkruiden   maggi  smaak</td>
    </tr>
    <tr>
      <th>448</th>
      <td>flapjes uit skala kalloni</td>
      <td>bladerdeegflapjes  jonge   gekookte ham     tomatenblokjes    italiaanse tuinkruiden   losgekl...</td>
    </tr>
    <tr>
      <th>449</th>
      <td>kabeljauw op vlaamse wijze</td>
      <td>kabeljauwmoten        laurier       roomboter   wit droog  zeezout</td>
    </tr>
    <tr>
      <th>450</th>
      <td>oerbrood met gerookte kip&amp;comma gorgonzola&amp;comma rucola en peer</td>
      <td>oerbrood  kruidenkaas naturel    dun   gorgonzola    peren geschild   partjes   rucola</td>
    </tr>
    <tr>
      <th>451</th>
      <td>tarbot en truffel verpakt in aardappelspaghetti</td>
      <td>truffels  snijbiet    kervel tomaat  visfumet  gevogeltebouillon  kalfsfond  koude     verg...</td>
    </tr>
    <tr>
      <th>452</th>
      <td>florida !!!</td>
      <td>jus dorange vruchtensap</td>
    </tr>
    <tr>
      <th>453</th>
      <td>briouat maa roz</td>
      <td>bstella   vulling  rijst   kaneelstokje  poedersuiker    gevliesde amandelen</td>
    </tr>
    <tr>
      <th>454</th>
      <td>italiaanse visschotel uit de oven</td>
      <td>kabeljauw    vis   per persoon gebruikt   gepelde   kruidenroomkaas boursin achtig  italiaanse...</td>
    </tr>
    <tr>
      <th>455</th>
      <td>rempah balletjes</td>
      <td>vlees       smaak  cocosnoot  gesnipperde   ketoembar  koenir   laos geraspt   frituurvet</td>
    </tr>
    <tr>
      <th>456</th>
      <td>vleisland se kaaswors</td>
      <td>lekker vet vleis  sout  koljander  growwe swart   fyn naeltjies  fyn neut  woestersous  chedda...</td>
    </tr>
    <tr>
      <th>457</th>
      <td>traktatie</td>
      <td>bananen ijslolliestokjes  pure chocolade    plantaardig wit bakvet folie</td>
    </tr>
    <tr>
      <th>458</th>
      <td>gefrituurd exotisch fruit met passievruchtencoulis</td>
      <td>mango papaja pitahaja  filodeeg passievruchten soeplepels suikerstroop eigeel bolletjes ijs</td>
    </tr>
    <tr>
      <th>459</th>
      <td>pom­poen-knol­sel­de­rij­stamp­pot met pad­den­stoe­len­jus</td>
      <td>kruimige   wittewijnazijn  margarine ah verspakket pompoen knolselderij stamppotgroente    kas...</td>
    </tr>
    <tr>
      <th>460</th>
      <td>chauffeursbowl</td>
      <td>sinaasappel   grenadine  citroenlimonade  sinaaslimonade ijsblokjes</td>
    </tr>
    <tr>
      <th>461</th>
      <td>pasta met cantharellen en truffelpesto</td>
      <td>pasta   spaghetti  cantharellen sjalotten      kookroom light      rucola   truffelpesto oil v...</td>
    </tr>
    <tr>
      <th>462</th>
      <td>[b]shake it baby[/b]</td>
      <td>yoghurt</td>
    </tr>
    <tr>
      <th>463</th>
      <td>kaviaar soezen</td>
      <td>deeg              vetten    bestuiven  vulling    kaviaar deense viskuit</td>
    </tr>
    <tr>
      <th>464</th>
      <td>salade met gerookte forel en grapefruit!</td>
      <td>grapefruits  gemengde salade  alfalfa  fijngeknipte  bislooksprieten   forelfilets   vinaigret...</td>
    </tr>
    <tr>
      <th>465</th>
      <td>aziatisch vissticks in een jasje van sesamzaad</td>
      <td>vis  visfilet  gember  lente    rijstwijn   sherry       frituurolie  sesamzaadjes    beslag n...</td>
    </tr>
    <tr>
      <th>466</th>
      <td>witte chocoladesouffleetjes</td>
      <td>poedersuiker   chocolade</td>
    </tr>
    <tr>
      <th>467</th>
      <td>varkensfilets in een jasje</td>
      <td>italiaanse   maizena  varkensfilets</td>
    </tr>
    <tr>
      <th>468</th>
      <td>kroatische bonensoep een heerlijke maaltijdsoep</td>
      <td>mager rookspek        gepeld     gepeld     schoongemaakt    laurierblad jeneverbessen geplet ...</td>
    </tr>
    <tr>
      <th>469</th>
      <td>gefrituurde groenten met tomatensaus</td>
      <td>patentbloem  volkorenbloem    chilipoeder  komijnpoeder djinten      bloemkool  roosjes auberg...</td>
    </tr>
    <tr>
      <th>470</th>
      <td>kabeljauwfilet in koriander-ketjapsaus</td>
      <td>kabeljauwfilet    sap      sjalotjes</td>
    </tr>
    <tr>
      <th>471</th>
      <td>zeebaarsfilet in basilicumsaus</td>
      <td>zeebaars  kabeljauwfilet       pijnboompitten  pesto   culinair  gesnipperde</td>
    </tr>
    <tr>
      <th>472</th>
      <td>geschnetzeltes</td>
      <td>kip  kalkoenfilet  kalfsvlees       meel     crème fraîche</td>
    </tr>
    <tr>
      <th>473</th>
      <td>schwarzwalder spätzlepfännle van gerôme &amp;period&amp;period&amp;period&amp;period&amp;period&amp;period</td>
      <td>kalkoenfilet   spätzle   kookroom     bakken   ragfijn      kippenbouillonblokje   garnering  ...</td>
    </tr>
    <tr>
      <th>474</th>
      <td>ananassorbet met munt</td>
      <td>rijpe ananas       munt  zoete   sap</td>
    </tr>
    <tr>
      <th>475</th>
      <td>gekookte mosselen met mosterd-biersaus</td>
      <td>mossel    schelp    selderij wortels   fles bier pils</td>
    </tr>
    <tr>
      <th>476</th>
      <td>andijviestamppot met perzik en shoarma</td>
      <td>kruimige aardappels   andijvie  perziken  shoarmavlees  vloeibare margarine   unox  magere roo...</td>
    </tr>
    <tr>
      <th>477</th>
      <td>dubbeldikke sandwich met haring</td>
      <td>schoongemaakte zoute haringen grof  sjalotjes   radijsjes grof   creme fraiche  mayonaise    vo...</td>
    </tr>
    <tr>
      <th>478</th>
      <td>muesli van haver met aardbeienmelk</td>
      <td>haver appel      zonnebloempitten  aardbeien   zure</td>
    </tr>
    <tr>
      <th>479</th>
      <td>kipgehaktbrood met ham en kwarteleitjes</td>
      <td>kwarteleitjes  witbrood  korst  mager kipgehakt   yorkham      kruiden     selderij    tuinkers</td>
    </tr>
    <tr>
      <th>480</th>
      <td>pitta-broodjes</td>
      <td>pitabroodjes     gerapte belegen  bosuitje      veldsla kerstomaatjes</td>
    </tr>
    <tr>
      <th>481</th>
      <td>karamello`s</td>
      <td>bodem       vulling    gecondenseerde   fijne kristalsuiker  pecannoten  pistachenoten garneren...</td>
    </tr>
    <tr>
      <th>482</th>
      <td>konijn in trappistenbier</td>
      <td>konijnenbouten  pruimen  trappistenbier  sjalotjes schaaltje juspoeder  warm     kruidnagel lau...</td>
    </tr>
    <tr>
      <th>483</th>
      <td>zebra's</td>
      <td>oude    roggebrood</td>
    </tr>
    <tr>
      <th>484</th>
      <td>kippenleverpate</td>
      <td>gesnipperde     margarine  kippenlevertjes     pistachenoten      cognac eidooiers      geple...</td>
    </tr>
    <tr>
      <th>485</th>
      <td>kipfilet a la rookvlees</td>
      <td>dubbele kipfilets  rookvlees ananaskaas  perzikkaas</td>
    </tr>
    <tr>
      <th>486</th>
      <td>gemberkoffie a la mij</td>
      <td>koffie</td>
    </tr>
    <tr>
      <th>487</th>
      <td>sinaasappel-suprise!</td>
      <td>jus dorange</td>
    </tr>
    <tr>
      <th>488</th>
      <td>warme salade van wintergroenten</td>
      <td>zoete aardappel wortelen pastinaken knolselderij bieten voorgekookt      walnoten   gekruimel...</td>
    </tr>
    <tr>
      <th>489</th>
      <td>komkommersalade met dillesaus</td>
      <td>komkommers  kropsla  dille        yoghurt</td>
    </tr>
    <tr>
      <th>490</th>
      <td>sjalottenbouillon</td>
      <td>sjalotten gepeld  maisolie  cognac    roquefort     bouillon</td>
    </tr>
    <tr>
      <th>491</th>
      <td>perzikenchutney</td>
      <td>goed rijpe perziken  rozijnen   poedersuiker  saffraan  wijnazijn  gemberpoeder  chilipoeder</td>
    </tr>
    <tr>
      <th>492</th>
      <td>non alcoholische pina colada milkshake</td>
      <td>cocos   ananas    ijs  vanille</td>
    </tr>
    <tr>
      <th>493</th>
      <td>riz casimir</td>
      <td>varkensfricandeau         pittige kerriesaus   ananasblokjes bananen</td>
    </tr>
    <tr>
      <th>494</th>
      <td>appelmuffins</td>
      <td>appels  kokosbloesemsuiker  vanillepoeder    biologische citroenschil  speltbloem  speltmeel  b...</td>
    </tr>
    <tr>
      <th>495</th>
      <td>gazpacho andaluz</td>
      <td>oud witbrood      wijnazijn  rijpe geurige  ontveld komkommer      knoflookteentjes    vierge ...</td>
    </tr>
    <tr>
      <th>496</th>
      <td>komkommersoep &amp;lparook lekker voor de feestdagen&amp;rpar</td>
      <td>komkommer gewassen  gedeeltelijk geschild    banen  grof          kippenbouillon    vegetarisc...</td>
    </tr>
    <tr>
      <th>497</th>
      <td>bisschopswijn:</td>
      <td></td>
    </tr>
    <tr>
      <th>498</th>
      <td>sinaasappel sunset</td>
      <td>jus dorange campari</td>
    </tr>
    <tr>
      <th>499</th>
      <td>kwee roti djawa</td>
      <td>rijstemeel  santen</td>
    </tr>
  </tbody>
</table>
</div>




```python
#FIND MOST COMMON SINGLE WORDS
```


```python
monogramvocabulary = Counter()
```


```python
monograms = rcp_c["ingrediënten"].str.split().apply(monogramvocabulary.update)
```


```python
monogram_freq = pd.DataFrame(monogramvocabulary.most_common(), columns = ['woord', 'freq'])
```


```python
monogram_freq[0:500]
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>woord</th>
      <th>freq</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>smaak</td>
      <td>7254</td>
    </tr>
    <tr>
      <th>1</th>
      <td>kruiden</td>
      <td>5544</td>
    </tr>
    <tr>
      <th>2</th>
      <td>sap</td>
      <td>5535</td>
    </tr>
    <tr>
      <th>3</th>
      <td>saus</td>
      <td>4241</td>
    </tr>
    <tr>
      <th>4</th>
      <td>creme</td>
      <td>4130</td>
    </tr>
    <tr>
      <th>5</th>
      <td>yoghurt</td>
      <td>4056</td>
    </tr>
    <tr>
      <th>6</th>
      <td>ham</td>
      <td>3927</td>
    </tr>
    <tr>
      <th>7</th>
      <td>basterdsuiker</td>
      <td>3862</td>
    </tr>
    <tr>
      <th>8</th>
      <td>mayonaise</td>
      <td>3848</td>
    </tr>
    <tr>
      <th>9</th>
      <td>gekookte</td>
      <td>3815</td>
    </tr>
    <tr>
      <th>10</th>
      <td>grof</td>
      <td>3799</td>
    </tr>
    <tr>
      <th>11</th>
      <td>fraiche</td>
      <td>3768</td>
    </tr>
    <tr>
      <th>12</th>
      <td>sambal</td>
      <td>3749</td>
    </tr>
    <tr>
      <th>13</th>
      <td>oregano</td>
      <td>3748</td>
    </tr>
    <tr>
      <th>14</th>
      <td>roomboter</td>
      <td>3730</td>
    </tr>
    <tr>
      <th>15</th>
      <td>rijst</td>
      <td>3725</td>
    </tr>
    <tr>
      <th>16</th>
      <td>poedersuiker</td>
      <td>3633</td>
    </tr>
    <tr>
      <th>17</th>
      <td>margarine</td>
      <td>3595</td>
    </tr>
    <tr>
      <th>18</th>
      <td>parmezaanse</td>
      <td>3593</td>
    </tr>
    <tr>
      <th>19</th>
      <td>gember</td>
      <td>3563</td>
    </tr>
    <tr>
      <th>20</th>
      <td>zonnebloemolie</td>
      <td>3518</td>
    </tr>
    <tr>
      <th>21</th>
      <td>kippenbouillon</td>
      <td>3513</td>
    </tr>
    <tr>
      <th>22</th>
      <td>zeezout</td>
      <td>3483</td>
    </tr>
    <tr>
      <th>23</th>
      <td>chocolade</td>
      <td>3454</td>
    </tr>
    <tr>
      <th>24</th>
      <td>gepelde</td>
      <td>3444</td>
    </tr>
    <tr>
      <th>25</th>
      <td>paneermeel</td>
      <td>3424</td>
    </tr>
    <tr>
      <th>26</th>
      <td>olijven</td>
      <td>3396</td>
    </tr>
    <tr>
      <th>27</th>
      <td>garnalen</td>
      <td>3353</td>
    </tr>
    <tr>
      <th>28</th>
      <td>bouillon</td>
      <td>3310</td>
    </tr>
    <tr>
      <th>29</th>
      <td>ringen</td>
      <td>3296</td>
    </tr>
    <tr>
      <th>30</th>
      <td>zure</td>
      <td>3231</td>
    </tr>
    <tr>
      <th>31</th>
      <td>komkommer</td>
      <td>3193</td>
    </tr>
    <tr>
      <th>32</th>
      <td>rozijnen</td>
      <td>3115</td>
    </tr>
    <tr>
      <th>33</th>
      <td>lente</td>
      <td>3075</td>
    </tr>
    <tr>
      <th>34</th>
      <td>dille</td>
      <td>2852</td>
    </tr>
    <tr>
      <th>35</th>
      <td>vruchtensap</td>
      <td>2839</td>
    </tr>
    <tr>
      <th>36</th>
      <td>uitjes</td>
      <td>2839</td>
    </tr>
    <tr>
      <th>37</th>
      <td>kerriepoeder</td>
      <td>2800</td>
    </tr>
    <tr>
      <th>38</th>
      <td>geschild</td>
      <td>2794</td>
    </tr>
    <tr>
      <th>39</th>
      <td>magere</td>
      <td>2788</td>
    </tr>
    <tr>
      <th>40</th>
      <td>sla</td>
      <td>2784</td>
    </tr>
    <tr>
      <th>41</th>
      <td>geraspt</td>
      <td>2766</td>
    </tr>
    <tr>
      <th>42</th>
      <td>rundergehakt</td>
      <td>2745</td>
    </tr>
    <tr>
      <th>43</th>
      <td>vanillesuiker</td>
      <td>2701</td>
    </tr>
    <tr>
      <th>44</th>
      <td>courgette</td>
      <td>2681</td>
    </tr>
    <tr>
      <th>45</th>
      <td>rozemarijn</td>
      <td>2680</td>
    </tr>
    <tr>
      <th>46</th>
      <td>bladerdeeg</td>
      <td>2669</td>
    </tr>
    <tr>
      <th>47</th>
      <td>molen</td>
      <td>2659</td>
    </tr>
    <tr>
      <th>48</th>
      <td>amandelen</td>
      <td>2637</td>
    </tr>
    <tr>
      <th>49</th>
      <td>zalm</td>
      <td>2637</td>
    </tr>
    <tr>
      <th>50</th>
      <td>zachte</td>
      <td>2617</td>
    </tr>
    <tr>
      <th>51</th>
      <td>rijpe</td>
      <td>2578</td>
    </tr>
    <tr>
      <th>52</th>
      <td>wijnazijn</td>
      <td>2576</td>
    </tr>
    <tr>
      <th>53</th>
      <td>cayennepeper</td>
      <td>2567</td>
    </tr>
    <tr>
      <th>54</th>
      <td>belegen</td>
      <td>2556</td>
    </tr>
    <tr>
      <th>55</th>
      <td>tomaat</td>
      <td>2535</td>
    </tr>
    <tr>
      <th>56</th>
      <td>aardappels</td>
      <td>2514</td>
    </tr>
    <tr>
      <th>57</th>
      <td>italiaanse</td>
      <td>2504</td>
    </tr>
    <tr>
      <th>58</th>
      <td>garnering</td>
      <td>2500</td>
    </tr>
    <tr>
      <th>59</th>
      <td>maizena</td>
      <td>2471</td>
    </tr>
    <tr>
      <th>60</th>
      <td>bleekselderij</td>
      <td>2465</td>
    </tr>
    <tr>
      <th>61</th>
      <td>kip</td>
      <td>2463</td>
    </tr>
    <tr>
      <th>62</th>
      <td>gesnipperde</td>
      <td>2434</td>
    </tr>
    <tr>
      <th>63</th>
      <td>sjalotjes</td>
      <td>2429</td>
    </tr>
    <tr>
      <th>64</th>
      <td>geperst</td>
      <td>2412</td>
    </tr>
    <tr>
      <th>65</th>
      <td>volle</td>
      <td>2405</td>
    </tr>
    <tr>
      <th>66</th>
      <td>spinazie</td>
      <td>2404</td>
    </tr>
    <tr>
      <th>67</th>
      <td>appel</td>
      <td>2398</td>
    </tr>
    <tr>
      <th>68</th>
      <td>bakpoeder</td>
      <td>2376</td>
    </tr>
    <tr>
      <th>69</th>
      <td>sherry</td>
      <td>2361</td>
    </tr>
    <tr>
      <th>70</th>
      <td>brood</td>
      <td>2350</td>
    </tr>
    <tr>
      <th>71</th>
      <td>gele</td>
      <td>2332</td>
    </tr>
    <tr>
      <th>72</th>
      <td>walnoten</td>
      <td>2331</td>
    </tr>
    <tr>
      <th>73</th>
      <td>pasta</td>
      <td>2318</td>
    </tr>
    <tr>
      <th>74</th>
      <td>gemengde</td>
      <td>2310</td>
    </tr>
    <tr>
      <th>75</th>
      <td>zoete</td>
      <td>2294</td>
    </tr>
    <tr>
      <th>76</th>
      <td>limoen</td>
      <td>2281</td>
    </tr>
    <tr>
      <th>77</th>
      <td>appels</td>
      <td>2261</td>
    </tr>
    <tr>
      <th>78</th>
      <td>manis</td>
      <td>2248</td>
    </tr>
    <tr>
      <th>79</th>
      <td>wortel</td>
      <td>2225</td>
    </tr>
    <tr>
      <th>80</th>
      <td>rum</td>
      <td>2207</td>
    </tr>
    <tr>
      <th>81</th>
      <td>fijne</td>
      <td>2167</td>
    </tr>
    <tr>
      <th>82</th>
      <td>asperges</td>
      <td>2156</td>
    </tr>
    <tr>
      <th>83</th>
      <td>munt</td>
      <td>2140</td>
    </tr>
    <tr>
      <th>84</th>
      <td>gepeld</td>
      <td>2136</td>
    </tr>
    <tr>
      <th>85</th>
      <td>schil</td>
      <td>2058</td>
    </tr>
    <tr>
      <th>86</th>
      <td>bakken</td>
      <td>2056</td>
    </tr>
    <tr>
      <th>87</th>
      <td>aardbeien</td>
      <td>2052</td>
    </tr>
    <tr>
      <th>88</th>
      <td>vanille</td>
      <td>2045</td>
    </tr>
    <tr>
      <th>89</th>
      <td>kappertjes</td>
      <td>2035</td>
    </tr>
    <tr>
      <th>90</th>
      <td>spek</td>
      <td>2028</td>
    </tr>
    <tr>
      <th>91</th>
      <td>vulling</td>
      <td>1971</td>
    </tr>
    <tr>
      <th>92</th>
      <td>kool</td>
      <td>1962</td>
    </tr>
    <tr>
      <th>93</th>
      <td>sinaasappel</td>
      <td>1961</td>
    </tr>
    <tr>
      <th>94</th>
      <td>bosuitjes</td>
      <td>1911</td>
    </tr>
    <tr>
      <th>95</th>
      <td>sjalotten</td>
      <td>1896</td>
    </tr>
    <tr>
      <th>96</th>
      <td>bonen</td>
      <td>1882</td>
    </tr>
    <tr>
      <th>97</th>
      <td>komijn</td>
      <td>1882</td>
    </tr>
    <tr>
      <th>98</th>
      <td>sjalot</td>
      <td>1852</td>
    </tr>
    <tr>
      <th>99</th>
      <td>roomkaas</td>
      <td>1851</td>
    </tr>
    <tr>
      <th>100</th>
      <td>cognac</td>
      <td>1846</td>
    </tr>
    <tr>
      <th>101</th>
      <td>ananas</td>
      <td>1846</td>
    </tr>
    <tr>
      <th>102</th>
      <td>keuze</td>
      <td>1831</td>
    </tr>
    <tr>
      <th>103</th>
      <td>gemberpoeder</td>
      <td>1820</td>
    </tr>
    <tr>
      <th>104</th>
      <td>wit</td>
      <td>1811</td>
    </tr>
    <tr>
      <th>105</th>
      <td>uitgelekt</td>
      <td>1801</td>
    </tr>
    <tr>
      <th>106</th>
      <td>poeder</td>
      <td>1794</td>
    </tr>
    <tr>
      <th>107</th>
      <td>chinese</td>
      <td>1784</td>
    </tr>
    <tr>
      <th>108</th>
      <td>pijnboompitten</td>
      <td>1777</td>
    </tr>
    <tr>
      <th>109</th>
      <td>gemberwortel</td>
      <td>1758</td>
    </tr>
    <tr>
      <th>110</th>
      <td>spaanse</td>
      <td>1739</td>
    </tr>
    <tr>
      <th>111</th>
      <td>pure</td>
      <td>1735</td>
    </tr>
    <tr>
      <th>112</th>
      <td>kerrie</td>
      <td>1726</td>
    </tr>
    <tr>
      <th>113</th>
      <td>koffie</td>
      <td>1716</td>
    </tr>
    <tr>
      <th>114</th>
      <td>pesto</td>
      <td>1714</td>
    </tr>
    <tr>
      <th>115</th>
      <td>bakmeel</td>
      <td>1701</td>
    </tr>
    <tr>
      <th>116</th>
      <td>gist</td>
      <td>1697</td>
    </tr>
    <tr>
      <th>117</th>
      <td>vloeibare</td>
      <td>1682</td>
    </tr>
    <tr>
      <th>118</th>
      <td>oude</td>
      <td>1677</td>
    </tr>
    <tr>
      <th>119</th>
      <td>zelfrijzend</td>
      <td>1664</td>
    </tr>
    <tr>
      <th>120</th>
      <td>kipfilets</td>
      <td>1657</td>
    </tr>
    <tr>
      <th>121</th>
      <td>mager</td>
      <td>1645</td>
    </tr>
    <tr>
      <th>122</th>
      <td>chilipoeder</td>
      <td>1627</td>
    </tr>
    <tr>
      <th>123</th>
      <td>broccoli</td>
      <td>1616</td>
    </tr>
    <tr>
      <th>124</th>
      <td>c</td>
      <td>1605</td>
    </tr>
    <tr>
      <th>125</th>
      <td>laurier</td>
      <td>1583</td>
    </tr>
    <tr>
      <th>126</th>
      <td>ontbijtspek</td>
      <td>1576</td>
    </tr>
    <tr>
      <th>127</th>
      <td>rucola</td>
      <td>1575</td>
    </tr>
    <tr>
      <th>128</th>
      <td>gesmolten</td>
      <td>1567</td>
    </tr>
    <tr>
      <th>129</th>
      <td>groentebouillon</td>
      <td>1560</td>
    </tr>
    <tr>
      <th>130</th>
      <td>sesamolie</td>
      <td>1544</td>
    </tr>
    <tr>
      <th>131</th>
      <td>sjalotje</td>
      <td>1540</td>
    </tr>
    <tr>
      <th>132</th>
      <td>kokosmelk</td>
      <td>1539</td>
    </tr>
    <tr>
      <th>133</th>
      <td>rauwe</td>
      <td>1532</td>
    </tr>
    <tr>
      <th>134</th>
      <td>mozzarella</td>
      <td>1532</td>
    </tr>
    <tr>
      <th>135</th>
      <td>eierdooiers</td>
      <td>1528</td>
    </tr>
    <tr>
      <th>136</th>
      <td>à</td>
      <td>1513</td>
    </tr>
    <tr>
      <th>137</th>
      <td>dressing</td>
      <td>1504</td>
    </tr>
    <tr>
      <th>138</th>
      <td>gelatine</td>
      <td>1500</td>
    </tr>
    <tr>
      <th>139</th>
      <td>crème</td>
      <td>1497</td>
    </tr>
    <tr>
      <th>140</th>
      <td>ringetjes</td>
      <td>1491</td>
    </tr>
    <tr>
      <th>141</th>
      <td>peperkorrels</td>
      <td>1491</td>
    </tr>
    <tr>
      <th>142</th>
      <td>pepers</td>
      <td>1472</td>
    </tr>
    <tr>
      <th>143</th>
      <td>laurierblaadjes</td>
      <td>1445</td>
    </tr>
    <tr>
      <th>144</th>
      <td>wortelen</td>
      <td>1436</td>
    </tr>
    <tr>
      <th>145</th>
      <td>pit</td>
      <td>1432</td>
    </tr>
    <tr>
      <th>146</th>
      <td>tonijn</td>
      <td>1417</td>
    </tr>
    <tr>
      <th>147</th>
      <td>laurierblad</td>
      <td>1417</td>
    </tr>
    <tr>
      <th>148</th>
      <td>dragon</td>
      <td>1415</td>
    </tr>
    <tr>
      <th>149</th>
      <td>geitenkaas</td>
      <td>1409</td>
    </tr>
    <tr>
      <th>150</th>
      <td>per</td>
      <td>1406</td>
    </tr>
    <tr>
      <th>151</th>
      <td>grove</td>
      <td>1405</td>
    </tr>
    <tr>
      <th>152</th>
      <td>kristalsuiker</td>
      <td>1400</td>
    </tr>
    <tr>
      <th>153</th>
      <td>gehalveerd</td>
      <td>1394</td>
    </tr>
    <tr>
      <th>154</th>
      <td>ongezouten</td>
      <td>1390</td>
    </tr>
    <tr>
      <th>155</th>
      <td>deeg</td>
      <td>1385</td>
    </tr>
    <tr>
      <th>156</th>
      <td>losgeklopt</td>
      <td>1380</td>
    </tr>
    <tr>
      <th>157</th>
      <td>rood</td>
      <td>1377</td>
    </tr>
    <tr>
      <th>158</th>
      <td>schoongemaakt</td>
      <td>1359</td>
    </tr>
    <tr>
      <th>159</th>
      <td>avocado</td>
      <td>1352</td>
    </tr>
    <tr>
      <th>160</th>
      <td>geroosterde</td>
      <td>1352</td>
    </tr>
    <tr>
      <th>161</th>
      <td>tabasco</td>
      <td>1351</td>
    </tr>
    <tr>
      <th>162</th>
      <td>balsamico</td>
      <td>1333</td>
    </tr>
    <tr>
      <th>163</th>
      <td>jonge</td>
      <td>1316</td>
    </tr>
    <tr>
      <th>164</th>
      <td>mosselen</td>
      <td>1313</td>
    </tr>
    <tr>
      <th>165</th>
      <td>eiwitten</td>
      <td>1306</td>
    </tr>
    <tr>
      <th>166</th>
      <td>kruidnagels</td>
      <td>1305</td>
    </tr>
    <tr>
      <th>167</th>
      <td>oelek</td>
      <td>1304</td>
    </tr>
    <tr>
      <th>168</th>
      <td>kwark</td>
      <td>1274</td>
    </tr>
    <tr>
      <th>169</th>
      <td>feta</td>
      <td>1274</td>
    </tr>
    <tr>
      <th>170</th>
      <td>mango</td>
      <td>1270</td>
    </tr>
    <tr>
      <th>171</th>
      <td>tomatenketchup</td>
      <td>1266</td>
    </tr>
    <tr>
      <th>172</th>
      <td>jus</td>
      <td>1255</td>
    </tr>
    <tr>
      <th>173</th>
      <td>liefst</td>
      <td>1251</td>
    </tr>
    <tr>
      <th>174</th>
      <td>sperziebonen</td>
      <td>1239</td>
    </tr>
    <tr>
      <th>175</th>
      <td>kurkuma</td>
      <td>1237</td>
    </tr>
    <tr>
      <th>176</th>
      <td>kervel</td>
      <td>1236</td>
    </tr>
    <tr>
      <th>177</th>
      <td>bananen</td>
      <td>1205</td>
    </tr>
    <tr>
      <th>178</th>
      <td>mix</td>
      <td>1203</td>
    </tr>
    <tr>
      <th>179</th>
      <td>mascarpone</td>
      <td>1198</td>
    </tr>
    <tr>
      <th>180</th>
      <td>bacon</td>
      <td>1191</td>
    </tr>
    <tr>
      <th>181</th>
      <td>spaghetti</td>
      <td>1190</td>
    </tr>
    <tr>
      <th>182</th>
      <td>gedroogd</td>
      <td>1188</td>
    </tr>
    <tr>
      <th>183</th>
      <td>eidooiers</td>
      <td>1186</td>
    </tr>
    <tr>
      <th>184</th>
      <td>look</td>
      <td>1181</td>
    </tr>
    <tr>
      <th>185</th>
      <td>hazelnoten</td>
      <td>1176</td>
    </tr>
    <tr>
      <th>186</th>
      <td>partjes</td>
      <td>1168</td>
    </tr>
    <tr>
      <th>187</th>
      <td>marinade</td>
      <td>1166</td>
    </tr>
    <tr>
      <th>188</th>
      <td>vlees</td>
      <td>1150</td>
    </tr>
    <tr>
      <th>189</th>
      <td>ketchup</td>
      <td>1149</td>
    </tr>
    <tr>
      <th>190</th>
      <td>gerookt</td>
      <td>1149</td>
    </tr>
    <tr>
      <th>191</th>
      <td>aardappel</td>
      <td>1146</td>
    </tr>
    <tr>
      <th>192</th>
      <td>koude</td>
      <td>1140</td>
    </tr>
    <tr>
      <th>193</th>
      <td>wodka</td>
      <td>1130</td>
    </tr>
    <tr>
      <th>194</th>
      <td>kant</td>
      <td>1125</td>
    </tr>
    <tr>
      <th>195</th>
      <td>vetten</td>
      <td>1122</td>
    </tr>
    <tr>
      <th>196</th>
      <td>vierge</td>
      <td>1119</td>
    </tr>
    <tr>
      <th>197</th>
      <td>sinaasappelsap</td>
      <td>1111</td>
    </tr>
    <tr>
      <th>198</th>
      <td>platte</td>
      <td>1104</td>
    </tr>
    <tr>
      <th>199</th>
      <td>vissaus</td>
      <td>1094</td>
    </tr>
    <tr>
      <th>200</th>
      <td>witlof</td>
      <td>1087</td>
    </tr>
    <tr>
      <th>201</th>
      <td>lichte</td>
      <td>1086</td>
    </tr>
    <tr>
      <th>202</th>
      <td>fraîche</td>
      <td>1085</td>
    </tr>
    <tr>
      <th>203</th>
      <td>kokos</td>
      <td>1083</td>
    </tr>
    <tr>
      <th>204</th>
      <td>fles</td>
      <td>1080</td>
    </tr>
    <tr>
      <th>205</th>
      <td>griekse</td>
      <td>1075</td>
    </tr>
    <tr>
      <th>206</th>
      <td>tomatenblokjes</td>
      <td>1075</td>
    </tr>
    <tr>
      <th>207</th>
      <td>bloemkool</td>
      <td>1071</td>
    </tr>
    <tr>
      <th>208</th>
      <td>gekookt</td>
      <td>1059</td>
    </tr>
    <tr>
      <th>209</th>
      <td>selderij</td>
      <td>1055</td>
    </tr>
    <tr>
      <th>210</th>
      <td>stevige</td>
      <td>1054</td>
    </tr>
    <tr>
      <th>211</th>
      <td>uitje</td>
      <td>1044</td>
    </tr>
    <tr>
      <th>212</th>
      <td>plantaardige</td>
      <td>1042</td>
    </tr>
    <tr>
      <th>213</th>
      <td>ah</td>
      <td>1042</td>
    </tr>
    <tr>
      <th>214</th>
      <td>salie</td>
      <td>1041</td>
    </tr>
    <tr>
      <th>215</th>
      <td>zaadjes</td>
      <td>1041</td>
    </tr>
    <tr>
      <th>216</th>
      <td>tomaatjes</td>
      <td>1028</td>
    </tr>
    <tr>
      <th>217</th>
      <td>kaneelpoeder</td>
      <td>1028</td>
    </tr>
    <tr>
      <th>218</th>
      <td>worteltjes</td>
      <td>1027</td>
    </tr>
    <tr>
      <th>219</th>
      <td>winterwortel</td>
      <td>1024</td>
    </tr>
    <tr>
      <th>220</th>
      <td>abrikozen</td>
      <td>1021</td>
    </tr>
    <tr>
      <th>221</th>
      <td>aubergine</td>
      <td>1020</td>
    </tr>
    <tr>
      <th>222</th>
      <td>limoensap</td>
      <td>1017</td>
    </tr>
    <tr>
      <th>223</th>
      <td>citroenen</td>
      <td>1015</td>
    </tr>
    <tr>
      <th>224</th>
      <td>preien</td>
      <td>1014</td>
    </tr>
    <tr>
      <th>225</th>
      <td>spekblokjes</td>
      <td>1011</td>
    </tr>
    <tr>
      <th>226</th>
      <td>conimex</td>
      <td>1009</td>
    </tr>
    <tr>
      <th>227</th>
      <td>ingrediënten</td>
      <td>995</td>
    </tr>
    <tr>
      <th>228</th>
      <td>knoflookpoeder</td>
      <td>994</td>
    </tr>
    <tr>
      <th>229</th>
      <td>frisdrank</td>
      <td>989</td>
    </tr>
    <tr>
      <th>230</th>
      <td>chilisaus</td>
      <td>989</td>
    </tr>
    <tr>
      <th>231</th>
      <td>gewassen</td>
      <td>988</td>
    </tr>
    <tr>
      <th>232</th>
      <td>maggi</td>
      <td>984</td>
    </tr>
    <tr>
      <th>233</th>
      <td>courgettes</td>
      <td>983</td>
    </tr>
    <tr>
      <th>234</th>
      <td>runderbouillon</td>
      <td>980</td>
    </tr>
    <tr>
      <th>235</th>
      <td>stokbrood</td>
      <td>977</td>
    </tr>
    <tr>
      <th>236</th>
      <td>zongedroogde</td>
      <td>973</td>
    </tr>
    <tr>
      <th>237</th>
      <td>eiwit</td>
      <td>968</td>
    </tr>
    <tr>
      <th>238</th>
      <td>ijsbergsla</td>
      <td>966</td>
    </tr>
    <tr>
      <th>239</th>
      <td>bv</td>
      <td>965</td>
    </tr>
    <tr>
      <th>240</th>
      <td>pepertje</td>
      <td>964</td>
    </tr>
    <tr>
      <th>241</th>
      <td>dorange</td>
      <td>961</td>
    </tr>
    <tr>
      <th>242</th>
      <td>ketoembar</td>
      <td>960</td>
    </tr>
    <tr>
      <th>243</th>
      <td>balsamicoazijn</td>
      <td>959</td>
    </tr>
    <tr>
      <th>244</th>
      <td>personen</td>
      <td>955</td>
    </tr>
    <tr>
      <th>245</th>
      <td>arachideolie</td>
      <td>954</td>
    </tr>
    <tr>
      <th>246</th>
      <td>wortels</td>
      <td>949</td>
    </tr>
    <tr>
      <th>247</th>
      <td>visbouillon</td>
      <td>949</td>
    </tr>
    <tr>
      <th>248</th>
      <td>bier</td>
      <td>947</td>
    </tr>
    <tr>
      <th>249</th>
      <td>kamertemperatuur</td>
      <td>947</td>
    </tr>
    <tr>
      <th>250</th>
      <td>gezeefde</td>
      <td>945</td>
    </tr>
    <tr>
      <th>251</th>
      <td>uitgeperst</td>
      <td>945</td>
    </tr>
    <tr>
      <th>252</th>
      <td>nodig</td>
      <td>944</td>
    </tr>
    <tr>
      <th>253</th>
      <td>olijf</td>
      <td>939</td>
    </tr>
    <tr>
      <th>254</th>
      <td>vleestomaten</td>
      <td>938</td>
    </tr>
    <tr>
      <th>255</th>
      <td>frambozen</td>
      <td>936</td>
    </tr>
    <tr>
      <th>256</th>
      <td>goed</td>
      <td>930</td>
    </tr>
    <tr>
      <th>257</th>
      <td>vastkokende</td>
      <td>926</td>
    </tr>
    <tr>
      <th>258</th>
      <td>banaan</td>
      <td>923</td>
    </tr>
    <tr>
      <th>259</th>
      <td>groen</td>
      <td>921</td>
    </tr>
    <tr>
      <th>260</th>
      <td>port</td>
      <td>917</td>
    </tr>
    <tr>
      <th>261</th>
      <td>laos</td>
      <td>916</td>
    </tr>
    <tr>
      <th>262</th>
      <td>geroosterd</td>
      <td>913</td>
    </tr>
    <tr>
      <th>263</th>
      <td>tauge</td>
      <td>904</td>
    </tr>
    <tr>
      <th>264</th>
      <td>mais</td>
      <td>903</td>
    </tr>
    <tr>
      <th>265</th>
      <td>tomatensaus</td>
      <td>903</td>
    </tr>
    <tr>
      <th>266</th>
      <td>eidooier</td>
      <td>902</td>
    </tr>
    <tr>
      <th>267</th>
      <td>vanillestokje</td>
      <td>901</td>
    </tr>
    <tr>
      <th>268</th>
      <td>vis</td>
      <td>897</td>
    </tr>
    <tr>
      <th>269</th>
      <td>gin</td>
      <td>893</td>
    </tr>
    <tr>
      <th>270</th>
      <td>lekker</td>
      <td>889</td>
    </tr>
    <tr>
      <th>271</th>
      <td>witbrood</td>
      <td>886</td>
    </tr>
    <tr>
      <th>272</th>
      <td>toko</td>
      <td>886</td>
    </tr>
    <tr>
      <th>273</th>
      <td>cashewnoten</td>
      <td>879</td>
    </tr>
    <tr>
      <th>274</th>
      <td>groenten</td>
      <td>873</td>
    </tr>
    <tr>
      <th>275</th>
      <td>eierdooier</td>
      <td>872</td>
    </tr>
    <tr>
      <th>276</th>
      <td>cacaopoeder</td>
      <td>853</td>
    </tr>
    <tr>
      <th>277</th>
      <td>hardgekookte</td>
      <td>852</td>
    </tr>
    <tr>
      <th>278</th>
      <td>doperwten</td>
      <td>843</td>
    </tr>
    <tr>
      <th>279</th>
      <td>grand</td>
      <td>843</td>
    </tr>
    <tr>
      <th>280</th>
      <td>spekreepjes</td>
      <td>842</td>
    </tr>
    <tr>
      <th>281</th>
      <td>kookroom</td>
      <td>840</td>
    </tr>
    <tr>
      <th>282</th>
      <td>fijngesnipperd</td>
      <td>837</td>
    </tr>
    <tr>
      <th>283</th>
      <td>sesamzaad</td>
      <td>835</td>
    </tr>
    <tr>
      <th>284</th>
      <td>augurken</td>
      <td>834</td>
    </tr>
    <tr>
      <th>285</th>
      <td>gembersiroop</td>
      <td>829</td>
    </tr>
    <tr>
      <th>286</th>
      <td>noten</td>
      <td>828</td>
    </tr>
    <tr>
      <th>287</th>
      <td>kabeljauw</td>
      <td>827</td>
    </tr>
    <tr>
      <th>288</th>
      <td>veldsla</td>
      <td>823</td>
    </tr>
    <tr>
      <th>289</th>
      <td>vleesbouillon</td>
      <td>822</td>
    </tr>
    <tr>
      <th>290</th>
      <td>goede</td>
      <td>815</td>
    </tr>
    <tr>
      <th>291</th>
      <td>kersen</td>
      <td>815</td>
    </tr>
    <tr>
      <th>292</th>
      <td>garneren</td>
      <td>814</td>
    </tr>
    <tr>
      <th>293</th>
      <td>rundvlees</td>
      <td>812</td>
    </tr>
    <tr>
      <th>294</th>
      <td>vergine</td>
      <td>812</td>
    </tr>
    <tr>
      <th>295</th>
      <td>naturel</td>
      <td>809</td>
    </tr>
    <tr>
      <th>296</th>
      <td>salade</td>
      <td>803</td>
    </tr>
    <tr>
      <th>297</th>
      <td>cacao</td>
      <td>799</td>
    </tr>
    <tr>
      <th>298</th>
      <td>ontveld</td>
      <td>798</td>
    </tr>
    <tr>
      <th>299</th>
      <td>hollandse</td>
      <td>796</td>
    </tr>
    <tr>
      <th>300</th>
      <td>blauwe</td>
      <td>796</td>
    </tr>
    <tr>
      <th>301</th>
      <td>tarwebloem</td>
      <td>795</td>
    </tr>
    <tr>
      <th>302</th>
      <td>kruimige</td>
      <td>794</td>
    </tr>
    <tr>
      <th>303</th>
      <td>kerstomaatjes</td>
      <td>793</td>
    </tr>
    <tr>
      <th>304</th>
      <td>verwijderd</td>
      <td>792</td>
    </tr>
    <tr>
      <th>305</th>
      <td>gebruik</td>
      <td>791</td>
    </tr>
    <tr>
      <th>306</th>
      <td>paddestoelen</td>
      <td>788</td>
    </tr>
    <tr>
      <th>307</th>
      <td>verkruimeld</td>
      <td>788</td>
    </tr>
    <tr>
      <th>308</th>
      <td>kopjes</td>
      <td>788</td>
    </tr>
    <tr>
      <th>309</th>
      <td>recept</td>
      <td>787</td>
    </tr>
    <tr>
      <th>310</th>
      <td>geschaafde</td>
      <td>785</td>
    </tr>
    <tr>
      <th>311</th>
      <td>fruit</td>
      <td>780</td>
    </tr>
    <tr>
      <th>312</th>
      <td>macaroni</td>
      <td>776</td>
    </tr>
    <tr>
      <th>313</th>
      <td>chilipeper</td>
      <td>775</td>
    </tr>
    <tr>
      <th>314</th>
      <td>zalmfilet</td>
      <td>773</td>
    </tr>
    <tr>
      <th>315</th>
      <td>pompoen</td>
      <td>766</td>
    </tr>
    <tr>
      <th>316</th>
      <td>bolletjes</td>
      <td>764</td>
    </tr>
    <tr>
      <th>317</th>
      <td>gebakken</td>
      <td>762</td>
    </tr>
    <tr>
      <th>318</th>
      <td>pittige</td>
      <td>762</td>
    </tr>
    <tr>
      <th>319</th>
      <td>rasp</td>
      <td>762</td>
    </tr>
    <tr>
      <th>320</th>
      <td>vel</td>
      <td>762</td>
    </tr>
    <tr>
      <th>321</th>
      <td>broodkruim</td>
      <td>758</td>
    </tr>
    <tr>
      <th>322</th>
      <td>pinda</td>
      <td>751</td>
    </tr>
    <tr>
      <th>323</th>
      <td>worcestersaus</td>
      <td>751</td>
    </tr>
    <tr>
      <th>324</th>
      <td>groente</td>
      <td>749</td>
    </tr>
    <tr>
      <th>325</th>
      <td>oud</td>
      <td>749</td>
    </tr>
    <tr>
      <th>326</th>
      <td>meel</td>
      <td>749</td>
    </tr>
    <tr>
      <th>327</th>
      <td>zuurkool</td>
      <td>748</td>
    </tr>
    <tr>
      <th>328</th>
      <td>kruidnagel</td>
      <td>746</td>
    </tr>
    <tr>
      <th>329</th>
      <td>peultjes</td>
      <td>745</td>
    </tr>
    <tr>
      <th>330</th>
      <td>komijnpoeder</td>
      <td>744</td>
    </tr>
    <tr>
      <th>331</th>
      <td>chili</td>
      <td>744</td>
    </tr>
    <tr>
      <th>332</th>
      <td>curry</td>
      <td>741</td>
    </tr>
    <tr>
      <th>333</th>
      <td>bouillonblokje</td>
      <td>736</td>
    </tr>
    <tr>
      <th>334</th>
      <td>tagliatelle</td>
      <td>735</td>
    </tr>
    <tr>
      <th>335</th>
      <td>kabeljauwfilet</td>
      <td>729</td>
    </tr>
    <tr>
      <th>336</th>
      <td>thee</td>
      <td>729</td>
    </tr>
    <tr>
      <th>337</th>
      <td>goudse</td>
      <td>728</td>
    </tr>
    <tr>
      <th>338</th>
      <td>peren</td>
      <td>726</td>
    </tr>
    <tr>
      <th>339</th>
      <td>perziken</td>
      <td>721</td>
    </tr>
    <tr>
      <th>340</th>
      <td>ontdooid</td>
      <td>721</td>
    </tr>
    <tr>
      <th>341</th>
      <td>ijs</td>
      <td>716</td>
    </tr>
    <tr>
      <th>342</th>
      <td>penne</td>
      <td>715</td>
    </tr>
    <tr>
      <th>343</th>
      <td>kastanjechampignons</td>
      <td>711</td>
    </tr>
    <tr>
      <th>344</th>
      <td>pindakaas</td>
      <td>711</td>
    </tr>
    <tr>
      <th>345</th>
      <td>chorizo</td>
      <td>708</td>
    </tr>
    <tr>
      <th>346</th>
      <td>druiven</td>
      <td>708</td>
    </tr>
    <tr>
      <th>347</th>
      <td>knolselderij</td>
      <td>704</td>
    </tr>
    <tr>
      <th>348</th>
      <td>ansjovisfilets</td>
      <td>703</td>
    </tr>
    <tr>
      <th>349</th>
      <td>gezouten</td>
      <td>702</td>
    </tr>
    <tr>
      <th>350</th>
      <td>kaneelstokje</td>
      <td>700</td>
    </tr>
    <tr>
      <th>351</th>
      <td>bruin</td>
      <td>697</td>
    </tr>
    <tr>
      <th>352</th>
      <td>djinten</td>
      <td>695</td>
    </tr>
    <tr>
      <th>353</th>
      <td>citroengras</td>
      <td>692</td>
    </tr>
    <tr>
      <th>354</th>
      <td>milde</td>
      <td>690</td>
    </tr>
    <tr>
      <th>355</th>
      <td>rietsuiker</td>
      <td>689</td>
    </tr>
    <tr>
      <th>356</th>
      <td>gorgonzola</td>
      <td>689</td>
    </tr>
    <tr>
      <th>357</th>
      <td>citroenschil</td>
      <td>686</td>
    </tr>
    <tr>
      <th>358</th>
      <td>japanse</td>
      <td>684</td>
    </tr>
    <tr>
      <th>359</th>
      <td>vet</td>
      <td>683</td>
    </tr>
    <tr>
      <th>360</th>
      <td>siroop</td>
      <td>682</td>
    </tr>
    <tr>
      <th>361</th>
      <td>ricotta</td>
      <td>680</td>
    </tr>
    <tr>
      <th>362</th>
      <td>sinaasappels</td>
      <td>680</td>
    </tr>
    <tr>
      <th>363</th>
      <td>waterkers</td>
      <td>680</td>
    </tr>
    <tr>
      <th>364</th>
      <td>pruimen</td>
      <td>678</td>
    </tr>
    <tr>
      <th>365</th>
      <td>salami</td>
      <td>676</td>
    </tr>
    <tr>
      <th>366</th>
      <td>blanke</td>
      <td>676</td>
    </tr>
    <tr>
      <th>367</th>
      <td>heet</td>
      <td>673</td>
    </tr>
    <tr>
      <th>368</th>
      <td>donkere</td>
      <td>673</td>
    </tr>
    <tr>
      <th>369</th>
      <td>sterke</td>
      <td>671</td>
    </tr>
    <tr>
      <th>370</th>
      <td>santen</td>
      <td>667</td>
    </tr>
    <tr>
      <th>371</th>
      <td>persoon</td>
      <td>663</td>
    </tr>
    <tr>
      <th>372</th>
      <td>bladpeterselie</td>
      <td>662</td>
    </tr>
    <tr>
      <th>373</th>
      <td>varkensvlees</td>
      <td>659</td>
    </tr>
    <tr>
      <th>374</th>
      <td>ingredienten</td>
      <td>658</td>
    </tr>
    <tr>
      <th>375</th>
      <td>maken</td>
      <td>655</td>
    </tr>
    <tr>
      <th>376</th>
      <td>mooie</td>
      <td>653</td>
    </tr>
    <tr>
      <th>377</th>
      <td>gebruikt</td>
      <td>652</td>
    </tr>
    <tr>
      <th>378</th>
      <td>provencaalse</td>
      <td>651</td>
    </tr>
    <tr>
      <th>379</th>
      <td>aardappelpuree</td>
      <td>651</td>
    </tr>
    <tr>
      <th>380</th>
      <td>geschilde</td>
      <td>650</td>
    </tr>
    <tr>
      <th>381</th>
      <td>oestersaus</td>
      <td>650</td>
    </tr>
    <tr>
      <th>382</th>
      <td>gaar</td>
      <td>647</td>
    </tr>
    <tr>
      <th>383</th>
      <td>blikken</td>
      <td>643</td>
    </tr>
    <tr>
      <th>384</th>
      <td>cointreau</td>
      <td>632</td>
    </tr>
    <tr>
      <th>385</th>
      <td>puree</td>
      <td>629</td>
    </tr>
    <tr>
      <th>386</th>
      <td>koud</td>
      <td>629</td>
    </tr>
    <tr>
      <th>387</th>
      <td>dik</td>
      <td>626</td>
    </tr>
    <tr>
      <th>388</th>
      <td>vermouth</td>
      <td>626</td>
    </tr>
    <tr>
      <th>389</th>
      <td>sereh</td>
      <td>624</td>
    </tr>
    <tr>
      <th>390</th>
      <td>frituren</td>
      <td>622</td>
    </tr>
    <tr>
      <th>391</th>
      <td>ontpit</td>
      <td>620</td>
    </tr>
    <tr>
      <th>392</th>
      <td>spekjes</td>
      <td>617</td>
    </tr>
    <tr>
      <th>393</th>
      <td>laurierblaadje</td>
      <td>617</td>
    </tr>
    <tr>
      <th>394</th>
      <td>aubergines</td>
      <td>616</td>
    </tr>
    <tr>
      <th>395</th>
      <td>tuinkruiden</td>
      <td>615</td>
    </tr>
    <tr>
      <th>396</th>
      <td>dun</td>
      <td>614</td>
    </tr>
    <tr>
      <th>397</th>
      <td>thaise</td>
      <td>613</td>
    </tr>
    <tr>
      <th>398</th>
      <td>vieren</td>
      <td>610</td>
    </tr>
    <tr>
      <th>399</th>
      <td>roze</td>
      <td>608</td>
    </tr>
    <tr>
      <th>400</th>
      <td>licht</td>
      <td>604</td>
    </tr>
    <tr>
      <th>401</th>
      <td>oesterzwammen</td>
      <td>604</td>
    </tr>
    <tr>
      <th>402</th>
      <td>sesamzaadjes</td>
      <td>600</td>
    </tr>
    <tr>
      <th>403</th>
      <td>v</td>
      <td>597</td>
    </tr>
    <tr>
      <th>404</th>
      <td>franse</td>
      <td>596</td>
    </tr>
    <tr>
      <th>405</th>
      <td>cheddar</td>
      <td>594</td>
    </tr>
    <tr>
      <th>406</th>
      <td>zilveruitjes</td>
      <td>588</td>
    </tr>
    <tr>
      <th>407</th>
      <td>champagne</td>
      <td>588</td>
    </tr>
    <tr>
      <th>408</th>
      <td>gezeefd</td>
      <td>585</td>
    </tr>
    <tr>
      <th>409</th>
      <td>warm</td>
      <td>580</td>
    </tr>
    <tr>
      <th>410</th>
      <td>panklare</td>
      <td>579</td>
    </tr>
    <tr>
      <th>411</th>
      <td>rookworst</td>
      <td>575</td>
    </tr>
    <tr>
      <th>412</th>
      <td>vingers</td>
      <td>574</td>
    </tr>
    <tr>
      <th>413</th>
      <td>whiskey</td>
      <td>574</td>
    </tr>
    <tr>
      <th>414</th>
      <td>n</td>
      <td>574</td>
    </tr>
    <tr>
      <th>415</th>
      <td>kleingesneden</td>
      <td>573</td>
    </tr>
    <tr>
      <th>416</th>
      <td>saffraan</td>
      <td>572</td>
    </tr>
    <tr>
      <th>417</th>
      <td>tuinkers</td>
      <td>570</td>
    </tr>
    <tr>
      <th>418</th>
      <td>extract</td>
      <td>569</td>
    </tr>
    <tr>
      <th>419</th>
      <td>winterpeen</td>
      <td>569</td>
    </tr>
    <tr>
      <th>420</th>
      <td>maïzena</td>
      <td>569</td>
    </tr>
    <tr>
      <th>421</th>
      <td>boursin</td>
      <td>568</td>
    </tr>
    <tr>
      <th>422</th>
      <td>kippebouillon</td>
      <td>568</td>
    </tr>
    <tr>
      <th>423</th>
      <td>korst</td>
      <td>566</td>
    </tr>
    <tr>
      <th>424</th>
      <td>gewone</td>
      <td>564</td>
    </tr>
    <tr>
      <th>425</th>
      <td>knoflookteentjes</td>
      <td>564</td>
    </tr>
    <tr>
      <th>426</th>
      <td>trassi</td>
      <td>559</td>
    </tr>
    <tr>
      <th>427</th>
      <td>koenjit</td>
      <td>559</td>
    </tr>
    <tr>
      <th>428</th>
      <td>kiwi</td>
      <td>557</td>
    </tr>
    <tr>
      <th>429</th>
      <td>trostomaten</td>
      <td>556</td>
    </tr>
    <tr>
      <th>430</th>
      <td>smalle</td>
      <td>556</td>
    </tr>
    <tr>
      <th>431</th>
      <td>krieltjes</td>
      <td>555</td>
    </tr>
    <tr>
      <th>432</th>
      <td>rijstwijn</td>
      <td>554</td>
    </tr>
    <tr>
      <th>433</th>
      <td>zoet</td>
      <td>554</td>
    </tr>
    <tr>
      <th>434</th>
      <td>ontpitte</td>
      <td>549</td>
    </tr>
    <tr>
      <th>435</th>
      <td>volkoren</td>
      <td>548</td>
    </tr>
    <tr>
      <th>436</th>
      <td>zaad</td>
      <td>548</td>
    </tr>
    <tr>
      <th>437</th>
      <td>gemengd</td>
      <td>547</td>
    </tr>
    <tr>
      <th>438</th>
      <td>linzen</td>
      <td>546</td>
    </tr>
    <tr>
      <th>439</th>
      <td>arachide</td>
      <td>542</td>
    </tr>
    <tr>
      <th>440</th>
      <td>basilicumblaadjes</td>
      <td>541</td>
    </tr>
    <tr>
      <th>441</th>
      <td>appelsap</td>
      <td>541</td>
    </tr>
    <tr>
      <th>442</th>
      <td>bessen</td>
      <td>539</td>
    </tr>
    <tr>
      <th>443</th>
      <td>jeneverbessen</td>
      <td>538</td>
    </tr>
    <tr>
      <th>444</th>
      <td>fijngeknipte</td>
      <td>537</td>
    </tr>
    <tr>
      <th>445</th>
      <td>laat</td>
      <td>536</td>
    </tr>
    <tr>
      <th>446</th>
      <td>jong</td>
      <td>534</td>
    </tr>
    <tr>
      <th>447</th>
      <td>medium</td>
      <td>533</td>
    </tr>
    <tr>
      <th>448</th>
      <td>halfvolle</td>
      <td>532</td>
    </tr>
    <tr>
      <th>449</th>
      <td>ontdaan</td>
      <td>532</td>
    </tr>
    <tr>
      <th>450</th>
      <td>soeplepels</td>
      <td>529</td>
    </tr>
    <tr>
      <th>451</th>
      <td>d</td>
      <td>527</td>
    </tr>
    <tr>
      <th>452</th>
      <td>mini</td>
      <td>526</td>
    </tr>
    <tr>
      <th>453</th>
      <td>geklopt</td>
      <td>523</td>
    </tr>
    <tr>
      <th>454</th>
      <td>roosjes</td>
      <td>522</td>
    </tr>
    <tr>
      <th>455</th>
      <td>schoongemaakte</td>
      <td>521</td>
    </tr>
    <tr>
      <th>456</th>
      <td>zoetzure</td>
      <td>521</td>
    </tr>
    <tr>
      <th>457</th>
      <td>likeur</td>
      <td>520</td>
    </tr>
    <tr>
      <th>458</th>
      <td>limoenen</td>
      <td>520</td>
    </tr>
    <tr>
      <th>459</th>
      <td>selder</td>
      <td>518</td>
    </tr>
    <tr>
      <th>460</th>
      <td>cherrytomaatjes</td>
      <td>514</td>
    </tr>
    <tr>
      <th>461</th>
      <td>piment</td>
      <td>511</td>
    </tr>
    <tr>
      <th>462</th>
      <td>erbij</td>
      <td>510</td>
    </tr>
    <tr>
      <th>463</th>
      <td>runder</td>
      <td>506</td>
    </tr>
    <tr>
      <th>464</th>
      <td>mierikswortel</td>
      <td>505</td>
    </tr>
    <tr>
      <th>465</th>
      <td>allesbinder</td>
      <td>505</td>
    </tr>
    <tr>
      <th>466</th>
      <td>selectie</td>
      <td>504</td>
    </tr>
    <tr>
      <th>467</th>
      <td>knoflooktenen</td>
      <td>502</td>
    </tr>
    <tr>
      <th>468</th>
      <td>dadels</td>
      <td>502</td>
    </tr>
    <tr>
      <th>469</th>
      <td>krenten</td>
      <td>502</td>
    </tr>
    <tr>
      <th>470</th>
      <td>harde</td>
      <td>501</td>
    </tr>
    <tr>
      <th>471</th>
      <td>masala</td>
      <td>501</td>
    </tr>
    <tr>
      <th>472</th>
      <td>light</td>
      <td>501</td>
    </tr>
    <tr>
      <th>473</th>
      <td>thijm</td>
      <td>500</td>
    </tr>
    <tr>
      <th>474</th>
      <td>koffieroom</td>
      <td>499</td>
    </tr>
    <tr>
      <th>475</th>
      <td>hete</td>
      <td>498</td>
    </tr>
    <tr>
      <th>476</th>
      <td>kruidenmix</td>
      <td>498</td>
    </tr>
    <tr>
      <th>477</th>
      <td>biefstuk</td>
      <td>496</td>
    </tr>
    <tr>
      <th>478</th>
      <td>geperste</td>
      <td>495</td>
    </tr>
    <tr>
      <th>479</th>
      <td>warme</td>
      <td>494</td>
    </tr>
    <tr>
      <th>480</th>
      <td>kruidnagelpoeder</td>
      <td>490</td>
    </tr>
    <tr>
      <th>481</th>
      <td>scampi</td>
      <td>489</td>
    </tr>
    <tr>
      <th>482</th>
      <td>parmezaan</td>
      <td>489</td>
    </tr>
    <tr>
      <th>483</th>
      <td>achterham</td>
      <td>489</td>
    </tr>
    <tr>
      <th>484</th>
      <td>mee</td>
      <td>489</td>
    </tr>
    <tr>
      <th>485</th>
      <td>bakboter</td>
      <td>489</td>
    </tr>
    <tr>
      <th>486</th>
      <td>spruitjes</td>
      <td>487</td>
    </tr>
    <tr>
      <th>487</th>
      <td>zonnebloem</td>
      <td>487</td>
    </tr>
    <tr>
      <th>488</th>
      <td>amandelschaafsel</td>
      <td>484</td>
    </tr>
    <tr>
      <th>489</th>
      <td>versgeraspte</td>
      <td>480</td>
    </tr>
    <tr>
      <th>490</th>
      <td>tomatensap</td>
      <td>478</td>
    </tr>
    <tr>
      <th>491</th>
      <td>marnier</td>
      <td>478</td>
    </tr>
    <tr>
      <th>492</th>
      <td>honig</td>
      <td>475</td>
    </tr>
    <tr>
      <th>493</th>
      <td>parmaham</td>
      <td>475</td>
    </tr>
    <tr>
      <th>494</th>
      <td>fijngesnipperde</td>
      <td>474</td>
    </tr>
    <tr>
      <th>495</th>
      <td>kikkererwten</td>
      <td>474</td>
    </tr>
    <tr>
      <th>496</th>
      <td>komijnzaad</td>
      <td>472</td>
    </tr>
    <tr>
      <th>497</th>
      <td>korianderpoeder</td>
      <td>471</td>
    </tr>
    <tr>
      <th>498</th>
      <td>vijgen</td>
      <td>471</td>
    </tr>
    <tr>
      <th>499</th>
      <td>marjolein</td>
      <td>470</td>
    </tr>
  </tbody>
</table>
</div>




```python
#FIND MOST COMMON BIGRAMS
```


```python
def return_bigrams(words):
    bigrams = zip(words, words[1:])
    return bigrams
```


```python
bigramvocabulary = Counter()
bigrams = rcp_c["ingrediënten"].str.split().apply(return_bigrams).apply(bigramvocabulary.update)
```


```python
bigram_freq = pd.DataFrame(bigramvocabulary.most_common(), columns = ['bigram', 'freq'])
```


```python
def concatenate_bigram(t):
    return str(t[0] + "_" + t[1])
```


```python
bigram_freq["bigram_conc"] = bigram_freq.bigram.apply(concatenate_bigram)
```


```python
bigram_freq[0:500]
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>bigram</th>
      <th>freq</th>
      <th>bigram_conc</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>(creme, fraiche)</td>
      <td>3427</td>
      <td>creme_fraiche</td>
    </tr>
    <tr>
      <th>1</th>
      <td>(lente, uitjes)</td>
      <td>1874</td>
      <td>lente_uitjes</td>
    </tr>
    <tr>
      <th>2</th>
      <td>(zelfrijzend, bakmeel)</td>
      <td>1610</td>
      <td>zelfrijzend_bakmeel</td>
    </tr>
    <tr>
      <th>3</th>
      <td>(pure, chocolade)</td>
      <td>1400</td>
      <td>pure_chocolade</td>
    </tr>
    <tr>
      <th>4</th>
      <td>(sambal, oelek)</td>
      <td>1270</td>
      <td>sambal_oelek</td>
    </tr>
    <tr>
      <th>5</th>
      <td>(italiaanse, kruiden)</td>
      <td>1188</td>
      <td>italiaanse_kruiden</td>
    </tr>
    <tr>
      <th>6</th>
      <td>(crème, fraîche)</td>
      <td>1035</td>
      <td>crème_fraîche</td>
    </tr>
    <tr>
      <th>7</th>
      <td>(jus, dorange)</td>
      <td>961</td>
      <td>jus_dorange</td>
    </tr>
    <tr>
      <th>8</th>
      <td>(olijven, pit)</td>
      <td>865</td>
      <td>olijven_pit</td>
    </tr>
    <tr>
      <th>9</th>
      <td>(gekookte, ham)</td>
      <td>813</td>
      <td>gekookte_ham</td>
    </tr>
    <tr>
      <th>10</th>
      <td>(rauwe, ham)</td>
      <td>790</td>
      <td>rauwe_ham</td>
    </tr>
    <tr>
      <th>11</th>
      <td>(griekse, yoghurt)</td>
      <td>742</td>
      <td>griekse_yoghurt</td>
    </tr>
    <tr>
      <th>12</th>
      <td>(grof, zeezout)</td>
      <td>627</td>
      <td>grof_zeezout</td>
    </tr>
    <tr>
      <th>13</th>
      <td>(per, persoon)</td>
      <td>620</td>
      <td>per_persoon</td>
    </tr>
    <tr>
      <th>14</th>
      <td>(geschaafde, amandelen)</td>
      <td>596</td>
      <td>geschaafde_amandelen</td>
    </tr>
    <tr>
      <th>15</th>
      <td>(gerookt, spek)</td>
      <td>595</td>
      <td>gerookt_spek</td>
    </tr>
    <tr>
      <th>16</th>
      <td>(sap, limoen)</td>
      <td>552</td>
      <td>sap_limoen</td>
    </tr>
    <tr>
      <th>17</th>
      <td>(gemengde, sla)</td>
      <td>549</td>
      <td>gemengde_sla</td>
    </tr>
    <tr>
      <th>18</th>
      <td>(provencaalse, kruiden)</td>
      <td>541</td>
      <td>provencaalse_kruiden</td>
    </tr>
    <tr>
      <th>19</th>
      <td>(c, selectie)</td>
      <td>500</td>
      <td>c_selectie</td>
    </tr>
    <tr>
      <th>20</th>
      <td>(magere, kwark)</td>
      <td>489</td>
      <td>magere_kwark</td>
    </tr>
    <tr>
      <th>21</th>
      <td>(vanille, extract)</td>
      <td>481</td>
      <td>vanille_extract</td>
    </tr>
    <tr>
      <th>22</th>
      <td>(zachte, geitenkaas)</td>
      <td>481</td>
      <td>zachte_geitenkaas</td>
    </tr>
    <tr>
      <th>23</th>
      <td>(rood, pepertje)</td>
      <td>481</td>
      <td>rood_pepertje</td>
    </tr>
    <tr>
      <th>24</th>
      <td>(grand, marnier)</td>
      <td>475</td>
      <td>grand_marnier</td>
    </tr>
    <tr>
      <th>25</th>
      <td>(gepelde, garnalen)</td>
      <td>465</td>
      <td>gepelde_garnalen</td>
    </tr>
    <tr>
      <th>26</th>
      <td>(jong, belegen)</td>
      <td>455</td>
      <td>jong_belegen</td>
    </tr>
    <tr>
      <th>27</th>
      <td>(kant, klare)</td>
      <td>443</td>
      <td>kant_klare</td>
    </tr>
    <tr>
      <th>28</th>
      <td>(chinese, kool)</td>
      <td>431</td>
      <td>chinese_kool</td>
    </tr>
    <tr>
      <th>29</th>
      <td>(fijne, kristalsuiker)</td>
      <td>421</td>
      <td>fijne_kristalsuiker</td>
    </tr>
    <tr>
      <th>30</th>
      <td>(magere, yoghurt)</td>
      <td>417</td>
      <td>magere_yoghurt</td>
    </tr>
    <tr>
      <th>31</th>
      <td>(hollandse, garnalen)</td>
      <td>395</td>
      <td>hollandse_garnalen</td>
    </tr>
    <tr>
      <th>32</th>
      <td>(rijpe, avocado)</td>
      <td>393</td>
      <td>rijpe_avocado</td>
    </tr>
    <tr>
      <th>33</th>
      <td>(spaanse, pepers)</td>
      <td>377</td>
      <td>spaanse_pepers</td>
    </tr>
    <tr>
      <th>34</th>
      <td>(sterke, koffie)</td>
      <td>371</td>
      <td>sterke_koffie</td>
    </tr>
    <tr>
      <th>35</th>
      <td>(wit, brood)</td>
      <td>357</td>
      <td>wit_brood</td>
    </tr>
    <tr>
      <th>36</th>
      <td>(garam, masala)</td>
      <td>349</td>
      <td>garam_masala</td>
    </tr>
    <tr>
      <th>37</th>
      <td>(parmezaanse, geraspt)</td>
      <td>344</td>
      <td>parmezaanse_geraspt</td>
    </tr>
    <tr>
      <th>38</th>
      <td>(volle, yoghurt)</td>
      <td>340</td>
      <td>volle_yoghurt</td>
    </tr>
    <tr>
      <th>39</th>
      <td>(vanille, ijs)</td>
      <td>331</td>
      <td>vanille_ijs</td>
    </tr>
    <tr>
      <th>40</th>
      <td>(grand, italia)</td>
      <td>324</td>
      <td>grand_italia</td>
    </tr>
    <tr>
      <th>41</th>
      <td>(zongedroogde, tomaatjes)</td>
      <td>318</td>
      <td>zongedroogde_tomaatjes</td>
    </tr>
    <tr>
      <th>42</th>
      <td>(magere, spekreepjes)</td>
      <td>317</td>
      <td>magere_spekreepjes</td>
    </tr>
    <tr>
      <th>43</th>
      <td>(traditioneel, c)</td>
      <td>316</td>
      <td>traditioneel_c</td>
    </tr>
    <tr>
      <th>44</th>
      <td>(granny, smith)</td>
      <td>306</td>
      <td>granny_smith</td>
    </tr>
    <tr>
      <th>45</th>
      <td>(sambal, badjak)</td>
      <td>305</td>
      <td>sambal_badjak</td>
    </tr>
    <tr>
      <th>46</th>
      <td>(rijpe, mango)</td>
      <td>302</td>
      <td>rijpe_mango</td>
    </tr>
    <tr>
      <th>47</th>
      <td>(gepelde, walnoten)</td>
      <td>297</td>
      <td>gepelde_walnoten</td>
    </tr>
    <tr>
      <th>48</th>
      <td>(vanille, essence)</td>
      <td>281</td>
      <td>vanille_essence</td>
    </tr>
    <tr>
      <th>49</th>
      <td>(kruimige, aardappels)</td>
      <td>280</td>
      <td>kruimige_aardappels</td>
    </tr>
    <tr>
      <th>50</th>
      <td>(mon, chou)</td>
      <td>280</td>
      <td>mon_chou</td>
    </tr>
    <tr>
      <th>51</th>
      <td>(gekookte, mosselen)</td>
      <td>277</td>
      <td>gekookte_mosselen</td>
    </tr>
    <tr>
      <th>52</th>
      <td>(mager, rundergehakt)</td>
      <td>274</td>
      <td>mager_rundergehakt</td>
    </tr>
    <tr>
      <th>53</th>
      <td>(ve, tsin)</td>
      <td>270</td>
      <td>ve_tsin</td>
    </tr>
    <tr>
      <th>54</th>
      <td>(shii, take)</td>
      <td>269</td>
      <td>shii_take</td>
    </tr>
    <tr>
      <th>55</th>
      <td>(crème, fraiche)</td>
      <td>267</td>
      <td>crème_fraiche</td>
    </tr>
    <tr>
      <th>56</th>
      <td>(sap, sinaasappel)</td>
      <td>266</td>
      <td>sap_sinaasappel</td>
    </tr>
    <tr>
      <th>57</th>
      <td>(smaak, smaak)</td>
      <td>260</td>
      <td>smaak_smaak</td>
    </tr>
    <tr>
      <th>58</th>
      <td>(ontpitte, olijven)</td>
      <td>259</td>
      <td>ontpitte_olijven</td>
    </tr>
    <tr>
      <th>59</th>
      <td>(lichtbruine, basterdsuiker)</td>
      <td>258</td>
      <td>lichtbruine_basterdsuiker</td>
    </tr>
    <tr>
      <th>60</th>
      <td>(broccoli, roosjes)</td>
      <td>258</td>
      <td>broccoli_roosjes</td>
    </tr>
    <tr>
      <th>61</th>
      <td>(belegen, goudse)</td>
      <td>256</td>
      <td>belegen_goudse</td>
    </tr>
    <tr>
      <th>62</th>
      <td>(gebakken, uitjes)</td>
      <td>252</td>
      <td>gebakken_uitjes</td>
    </tr>
    <tr>
      <th>63</th>
      <td>(goede, kwaliteit)</td>
      <td>247</td>
      <td>goede_kwaliteit</td>
    </tr>
    <tr>
      <th>64</th>
      <td>(magere, spekblokjes)</td>
      <td>247</td>
      <td>magere_spekblokjes</td>
    </tr>
    <tr>
      <th>65</th>
      <td>(bittere, chocolade)</td>
      <td>245</td>
      <td>bittere_chocolade</td>
    </tr>
    <tr>
      <th>66</th>
      <td>(zeezout, molen)</td>
      <td>245</td>
      <td>zeezout_molen</td>
    </tr>
    <tr>
      <th>67</th>
      <td>(vastkokende, aardappels)</td>
      <td>245</td>
      <td>vastkokende_aardappels</td>
    </tr>
    <tr>
      <th>68</th>
      <td>(gekookte, rijst)</td>
      <td>244</td>
      <td>gekookte_rijst</td>
    </tr>
    <tr>
      <th>69</th>
      <td>(gemengde, paddestoelen)</td>
      <td>242</td>
      <td>gemengde_paddestoelen</td>
    </tr>
    <tr>
      <th>70</th>
      <td>(aardappels, geschild)</td>
      <td>241</td>
      <td>aardappels_geschild</td>
    </tr>
    <tr>
      <th>71</th>
      <td>(roomkaas, kruiden)</td>
      <td>241</td>
      <td>roomkaas_kruiden</td>
    </tr>
    <tr>
      <th>72</th>
      <td>(zachte, roomboter)</td>
      <td>240</td>
      <td>zachte_roomboter</td>
    </tr>
    <tr>
      <th>73</th>
      <td>(spaans, pepertje)</td>
      <td>238</td>
      <td>spaans_pepertje</td>
    </tr>
    <tr>
      <th>74</th>
      <td>(manis, sambal)</td>
      <td>238</td>
      <td>manis_sambal</td>
    </tr>
    <tr>
      <th>75</th>
      <td>(zoete, chilisaus)</td>
      <td>229</td>
      <td>zoete_chilisaus</td>
    </tr>
    <tr>
      <th>76</th>
      <td>(bosuitjes, ringetjes)</td>
      <td>229</td>
      <td>bosuitjes_ringetjes</td>
    </tr>
    <tr>
      <th>77</th>
      <td>(rijstwijn, sherry)</td>
      <td>227</td>
      <td>rijstwijn_sherry</td>
    </tr>
    <tr>
      <th>78</th>
      <td>(rijpe, bananen)</td>
      <td>225</td>
      <td>rijpe_bananen</td>
    </tr>
    <tr>
      <th>79</th>
      <td>(bouquet, garni)</td>
      <td>224</td>
      <td>bouquet_garni</td>
    </tr>
    <tr>
      <th>80</th>
      <td>(kruiden, smaak)</td>
      <td>222</td>
      <td>kruiden_smaak</td>
    </tr>
    <tr>
      <th>81</th>
      <td>(cherry, tomaatjes)</td>
      <td>222</td>
      <td>cherry_tomaatjes</td>
    </tr>
    <tr>
      <th>82</th>
      <td>(oud, brood)</td>
      <td>221</td>
      <td>oud_brood</td>
    </tr>
    <tr>
      <th>83</th>
      <td>(schil, sinaasappel)</td>
      <td>219</td>
      <td>schil_sinaasappel</td>
    </tr>
    <tr>
      <th>84</th>
      <td>(schil, sap)</td>
      <td>218</td>
      <td>schil_sap</td>
    </tr>
    <tr>
      <th>85</th>
      <td>(sap, citroenen)</td>
      <td>217</td>
      <td>sap_citroenen</td>
    </tr>
    <tr>
      <th>86</th>
      <td>(basterdsuiker, vanillesuiker)</td>
      <td>217</td>
      <td>basterdsuiker_vanillesuiker</td>
    </tr>
    <tr>
      <th>87</th>
      <td>(gepelde, amandelen)</td>
      <td>216</td>
      <td>gepelde_amandelen</td>
    </tr>
    <tr>
      <th>88</th>
      <td>(rum, vruchtensap)</td>
      <td>215</td>
      <td>rum_vruchtensap</td>
    </tr>
    <tr>
      <th>89</th>
      <td>(hartige, taart)</td>
      <td>209</td>
      <td>hartige_taart</td>
    </tr>
    <tr>
      <th>90</th>
      <td>(geroosterde, pijnboompitten)</td>
      <td>208</td>
      <td>geroosterde_pijnboompitten</td>
    </tr>
    <tr>
      <th>91</th>
      <td>(ongezouten, roomboter)</td>
      <td>208</td>
      <td>ongezouten_roomboter</td>
    </tr>
    <tr>
      <th>92</th>
      <td>(lente, uitje)</td>
      <td>208</td>
      <td>lente_uitje</td>
    </tr>
    <tr>
      <th>93</th>
      <td>(gember, geraspt)</td>
      <td>207</td>
      <td>gember_geraspt</td>
    </tr>
    <tr>
      <th>94</th>
      <td>(blanke, amandelen)</td>
      <td>207</td>
      <td>blanke_amandelen</td>
    </tr>
    <tr>
      <th>95</th>
      <td>(vloeibare, margarine)</td>
      <td>206</td>
      <td>vloeibare_margarine</td>
    </tr>
    <tr>
      <th>96</th>
      <td>(opgelost, heet)</td>
      <td>206</td>
      <td>opgelost_heet</td>
    </tr>
    <tr>
      <th>97</th>
      <td>(mager, rookspek)</td>
      <td>206</td>
      <td>mager_rookspek</td>
    </tr>
    <tr>
      <th>98</th>
      <td>(blauwe, bessen)</td>
      <td>205</td>
      <td>blauwe_bessen</td>
    </tr>
    <tr>
      <th>99</th>
      <td>(blanke, rozijnen)</td>
      <td>199</td>
      <td>blanke_rozijnen</td>
    </tr>
    <tr>
      <th>100</th>
      <td>(pasta, keuze)</td>
      <td>199</td>
      <td>pasta_keuze</td>
    </tr>
    <tr>
      <th>101</th>
      <td>(grijze, garnalen)</td>
      <td>198</td>
      <td>grijze_garnalen</td>
    </tr>
    <tr>
      <th>102</th>
      <td>(doorregen, spek)</td>
      <td>197</td>
      <td>doorregen_spek</td>
    </tr>
    <tr>
      <th>103</th>
      <td>(gemengde, kruiden)</td>
      <td>196</td>
      <td>gemengde_kruiden</td>
    </tr>
    <tr>
      <th>104</th>
      <td>(komijn, djinten)</td>
      <td>195</td>
      <td>komijn_djinten</td>
    </tr>
    <tr>
      <th>105</th>
      <td>(donkerbruine, basterdsuiker)</td>
      <td>192</td>
      <td>donkerbruine_basterdsuiker</td>
    </tr>
    <tr>
      <th>106</th>
      <td>(roze, garnalen)</td>
      <td>190</td>
      <td>roze_garnalen</td>
    </tr>
    <tr>
      <th>107</th>
      <td>(hard, gekookte)</td>
      <td>186</td>
      <td>hard_gekookte</td>
    </tr>
    <tr>
      <th>108</th>
      <td>(zoete, aardappel)</td>
      <td>185</td>
      <td>zoete_aardappel</td>
    </tr>
    <tr>
      <th>109</th>
      <td>(haricots, verts)</td>
      <td>184</td>
      <td>haricots_verts</td>
    </tr>
    <tr>
      <th>110</th>
      <td>(pisang, ambon)</td>
      <td>183</td>
      <td>pisang_ambon</td>
    </tr>
    <tr>
      <th>111</th>
      <td>(bulgaarse, yoghurt)</td>
      <td>183</td>
      <td>bulgaarse_yoghurt</td>
    </tr>
    <tr>
      <th>112</th>
      <td>(thaise, vissaus)</td>
      <td>182</td>
      <td>thaise_vissaus</td>
    </tr>
    <tr>
      <th>113</th>
      <td>(fijne, tafelsuiker)</td>
      <td>181</td>
      <td>fijne_tafelsuiker</td>
    </tr>
    <tr>
      <th>114</th>
      <td>(bladerdeeg, ontdooid)</td>
      <td>180</td>
      <td>bladerdeeg_ontdooid</td>
    </tr>
    <tr>
      <th>115</th>
      <td>(grof, geraspt)</td>
      <td>179</td>
      <td>grof_geraspt</td>
    </tr>
    <tr>
      <th>116</th>
      <td>(mayonaise, yoghurt)</td>
      <td>179</td>
      <td>mayonaise_yoghurt</td>
    </tr>
    <tr>
      <th>117</th>
      <td>(garnalen, gepeld)</td>
      <td>178</td>
      <td>garnalen_gepeld</td>
    </tr>
    <tr>
      <th>118</th>
      <td>(nam, pla)</td>
      <td>177</td>
      <td>nam_pla</td>
    </tr>
    <tr>
      <th>119</th>
      <td>(medium, dry)</td>
      <td>177</td>
      <td>medium_dry</td>
    </tr>
    <tr>
      <th>120</th>
      <td>(limoen, sap)</td>
      <td>176</td>
      <td>limoen_sap</td>
    </tr>
    <tr>
      <th>121</th>
      <td>(aubergine, courgette)</td>
      <td>175</td>
      <td>aubergine_courgette</td>
    </tr>
    <tr>
      <th>122</th>
      <td>(conimex, wok)</td>
      <td>174</td>
      <td>conimex_wok</td>
    </tr>
    <tr>
      <th>123</th>
      <td>(vruchtensap, vruchtensap)</td>
      <td>174</td>
      <td>vruchtensap_vruchtensap</td>
    </tr>
    <tr>
      <th>124</th>
      <td>(bloemkool, roosjes)</td>
      <td>171</td>
      <td>bloemkool_roosjes</td>
    </tr>
    <tr>
      <th>125</th>
      <td>(zure, appels)</td>
      <td>170</td>
      <td>zure_appels</td>
    </tr>
    <tr>
      <th>126</th>
      <td>(vruchtensap, frisdrank)</td>
      <td>169</td>
      <td>vruchtensap_frisdrank</td>
    </tr>
    <tr>
      <th>127</th>
      <td>(donkere, basterdsuiker)</td>
      <td>166</td>
      <td>donkere_basterdsuiker</td>
    </tr>
    <tr>
      <th>128</th>
      <td>(ras, hanout)</td>
      <td>166</td>
      <td>ras_hanout</td>
    </tr>
    <tr>
      <th>129</th>
      <td>(stronkjes, witlof)</td>
      <td>166</td>
      <td>stronkjes_witlof</td>
    </tr>
    <tr>
      <th>130</th>
      <td>(gember, geschild)</td>
      <td>166</td>
      <td>gember_geschild</td>
    </tr>
    <tr>
      <th>131</th>
      <td>(ma, zena)</td>
      <td>165</td>
      <td>ma_zena</td>
    </tr>
    <tr>
      <th>132</th>
      <td>(oude, goudse)</td>
      <td>163</td>
      <td>oude_goudse</td>
    </tr>
    <tr>
      <th>133</th>
      <td>(gepeld, gepeld)</td>
      <td>163</td>
      <td>gepeld_gepeld</td>
    </tr>
    <tr>
      <th>134</th>
      <td>(bonen, tomatensaus)</td>
      <td>163</td>
      <td>bonen_tomatensaus</td>
    </tr>
    <tr>
      <th>135</th>
      <td>(yoghurt, mayonaise)</td>
      <td>161</td>
      <td>yoghurt_mayonaise</td>
    </tr>
    <tr>
      <th>136</th>
      <td>(sap, rasp)</td>
      <td>159</td>
      <td>sap_rasp</td>
    </tr>
    <tr>
      <th>137</th>
      <td>(kruiden, oregano)</td>
      <td>159</td>
      <td>kruiden_oregano</td>
    </tr>
    <tr>
      <th>138</th>
      <td>(blauwe, druiven)</td>
      <td>158</td>
      <td>blauwe_druiven</td>
    </tr>
    <tr>
      <th>139</th>
      <td>(dry, sherry)</td>
      <td>157</td>
      <td>dry_sherry</td>
    </tr>
    <tr>
      <th>140</th>
      <td>(italiaanse, keukenkruiden)</td>
      <td>155</td>
      <td>italiaanse_keukenkruiden</td>
    </tr>
    <tr>
      <th>141</th>
      <td>(courgette, aubergine)</td>
      <td>155</td>
      <td>courgette_aubergine</td>
    </tr>
    <tr>
      <th>142</th>
      <td>(mager, varkensvlees)</td>
      <td>155</td>
      <td>mager_varkensvlees</td>
    </tr>
    <tr>
      <th>143</th>
      <td>(gemengde, noten)</td>
      <td>155</td>
      <td>gemengde_noten</td>
    </tr>
    <tr>
      <th>144</th>
      <td>(lollo, rosso)</td>
      <td>154</td>
      <td>lollo_rosso</td>
    </tr>
    <tr>
      <th>145</th>
      <td>(rauwe, garnalen)</td>
      <td>152</td>
      <td>rauwe_garnalen</td>
    </tr>
    <tr>
      <th>146</th>
      <td>(zaadjes, verwijderd)</td>
      <td>152</td>
      <td>zaadjes_verwijderd</td>
    </tr>
    <tr>
      <th>147</th>
      <td>(mager, spek)</td>
      <td>152</td>
      <td>mager_spek</td>
    </tr>
    <tr>
      <th>148</th>
      <td>(brood, korst)</td>
      <td>151</td>
      <td>brood_korst</td>
    </tr>
    <tr>
      <th>149</th>
      <td>(rozijnen, krenten)</td>
      <td>151</td>
      <td>rozijnen_krenten</td>
    </tr>
    <tr>
      <th>150</th>
      <td>(sambal, manis)</td>
      <td>151</td>
      <td>sambal_manis</td>
    </tr>
    <tr>
      <th>151</th>
      <td>(provençaalse, kruiden)</td>
      <td>150</td>
      <td>provençaalse_kruiden</td>
    </tr>
    <tr>
      <th>152</th>
      <td>(olijven, ontpit)</td>
      <td>149</td>
      <td>olijven_ontpit</td>
    </tr>
    <tr>
      <th>153</th>
      <td>(blikken, gepelde)</td>
      <td>148</td>
      <td>blikken_gepelde</td>
    </tr>
    <tr>
      <th>154</th>
      <td>(bakmeel, bakpoeder)</td>
      <td>145</td>
      <td>bakmeel_bakpoeder</td>
    </tr>
    <tr>
      <th>155</th>
      <td>(lekker, vindt)</td>
      <td>145</td>
      <td>lekker_vindt</td>
    </tr>
    <tr>
      <th>156</th>
      <td>(rol, bladerdeeg)</td>
      <td>144</td>
      <td>rol_bladerdeeg</td>
    </tr>
    <tr>
      <th>157</th>
      <td>(mager, gerookt)</td>
      <td>144</td>
      <td>mager_gerookt</td>
    </tr>
    <tr>
      <th>158</th>
      <td>(bolletje, mozzarella)</td>
      <td>144</td>
      <td>bolletje_mozzarella</td>
    </tr>
    <tr>
      <th>159</th>
      <td>(ongezouten, cashewnoten)</td>
      <td>144</td>
      <td>ongezouten_cashewnoten</td>
    </tr>
    <tr>
      <th>160</th>
      <td>(bruin, brood)</td>
      <td>143</td>
      <td>bruin_brood</td>
    </tr>
    <tr>
      <th>161</th>
      <td>(basmati, rijst)</td>
      <td>142</td>
      <td>basmati_rijst</td>
    </tr>
    <tr>
      <th>162</th>
      <td>(gemengde, salade)</td>
      <td>141</td>
      <td>gemengde_salade</td>
    </tr>
    <tr>
      <th>163</th>
      <td>(witbrood, korst)</td>
      <td>141</td>
      <td>witbrood_korst</td>
    </tr>
    <tr>
      <th>164</th>
      <td>(versgeraspte, parmezaanse)</td>
      <td>141</td>
      <td>versgeraspte_parmezaanse</td>
    </tr>
    <tr>
      <th>165</th>
      <td>(goela, djawa)</td>
      <td>140</td>
      <td>goela_djawa</td>
    </tr>
    <tr>
      <th>166</th>
      <td>(grana, padano)</td>
      <td>140</td>
      <td>grana_padano</td>
    </tr>
    <tr>
      <th>167</th>
      <td>(uitjes, ringetjes)</td>
      <td>140</td>
      <td>uitjes_ringetjes</td>
    </tr>
    <tr>
      <th>168</th>
      <td>(zure, appel)</td>
      <td>139</td>
      <td>zure_appel</td>
    </tr>
    <tr>
      <th>169</th>
      <td>(gevulde, olijven)</td>
      <td>139</td>
      <td>gevulde_olijven</td>
    </tr>
    <tr>
      <th>170</th>
      <td>(struik, bleekselderij)</td>
      <td>138</td>
      <td>struik_bleekselderij</td>
    </tr>
    <tr>
      <th>171</th>
      <td>(krenten, rozijnen)</td>
      <td>138</td>
      <td>krenten_rozijnen</td>
    </tr>
    <tr>
      <th>172</th>
      <td>(zaadlijsten, verwijderd)</td>
      <td>138</td>
      <td>zaadlijsten_verwijderd</td>
    </tr>
    <tr>
      <th>173</th>
      <td>(noorse, garnalen)</td>
      <td>138</td>
      <td>noorse_garnalen</td>
    </tr>
    <tr>
      <th>174</th>
      <td>(magere, runderlappen)</td>
      <td>138</td>
      <td>magere_runderlappen</td>
    </tr>
    <tr>
      <th>175</th>
      <td>(sambal, smaak)</td>
      <td>138</td>
      <td>sambal_smaak</td>
    </tr>
    <tr>
      <th>176</th>
      <td>(oud, witbrood)</td>
      <td>137</td>
      <td>oud_witbrood</td>
    </tr>
    <tr>
      <th>177</th>
      <td>(geschild, geraspt)</td>
      <td>137</td>
      <td>geschild_geraspt</td>
    </tr>
    <tr>
      <th>178</th>
      <td>(gemengde, paddenstoelen)</td>
      <td>135</td>
      <td>gemengde_paddenstoelen</td>
    </tr>
    <tr>
      <th>179</th>
      <td>(wit, casinobrood)</td>
      <td>134</td>
      <td>wit_casinobrood</td>
    </tr>
    <tr>
      <th>180</th>
      <td>(gekookte, garnalen)</td>
      <td>134</td>
      <td>gekookte_garnalen</td>
    </tr>
    <tr>
      <th>181</th>
      <td>(ongezouten, pinda)</td>
      <td>134</td>
      <td>ongezouten_pinda</td>
    </tr>
    <tr>
      <th>182</th>
      <td>(appels, geschild)</td>
      <td>134</td>
      <td>appels_geschild</td>
    </tr>
    <tr>
      <th>183</th>
      <td>(sap, schil)</td>
      <td>133</td>
      <td>sap_schil</td>
    </tr>
    <tr>
      <th>184</th>
      <td>(deeg, hartige)</td>
      <td>131</td>
      <td>deeg_hartige</td>
    </tr>
    <tr>
      <th>185</th>
      <td>(jonnie, boer)</td>
      <td>131</td>
      <td>jonnie_boer</td>
    </tr>
    <tr>
      <th>186</th>
      <td>(stevige, appels)</td>
      <td>130</td>
      <td>stevige_appels</td>
    </tr>
    <tr>
      <th>187</th>
      <td>(kerrie, poeder)</td>
      <td>129</td>
      <td>kerrie_poeder</td>
    </tr>
    <tr>
      <th>188</th>
      <td>(gemberwortel, geschild)</td>
      <td>129</td>
      <td>gemberwortel_geschild</td>
    </tr>
    <tr>
      <th>189</th>
      <td>(uitjes, ringen)</td>
      <td>128</td>
      <td>uitjes_ringen</td>
    </tr>
    <tr>
      <th>190</th>
      <td>(gesnipperde, gesnipperde)</td>
      <td>127</td>
      <td>gesnipperde_gesnipperde</td>
    </tr>
    <tr>
      <th>191</th>
      <td>(lekker, vind)</td>
      <td>127</td>
      <td>lekker_vind</td>
    </tr>
    <tr>
      <th>192</th>
      <td>(los, geklopt)</td>
      <td>127</td>
      <td>los_geklopt</td>
    </tr>
    <tr>
      <th>193</th>
      <td>(gist, gist)</td>
      <td>127</td>
      <td>gist_gist</td>
    </tr>
    <tr>
      <th>194</th>
      <td>(gerookt, ontbijtspek)</td>
      <td>127</td>
      <td>gerookt_ontbijtspek</td>
    </tr>
    <tr>
      <th>195</th>
      <td>(walnoten, grof)</td>
      <td>126</td>
      <td>walnoten_grof</td>
    </tr>
    <tr>
      <th>196</th>
      <td>(zoetzure, augurken)</td>
      <td>126</td>
      <td>zoetzure_augurken</td>
    </tr>
    <tr>
      <th>197</th>
      <td>(soja, saus)</td>
      <td>125</td>
      <td>soja_saus</td>
    </tr>
    <tr>
      <th>198</th>
      <td>(wodka, vruchtensap)</td>
      <td>125</td>
      <td>wodka_vruchtensap</td>
    </tr>
    <tr>
      <th>199</th>
      <td>(rucola, sla)</td>
      <td>124</td>
      <td>rucola_sla</td>
    </tr>
    <tr>
      <th>200</th>
      <td>(oregano, rozemarijn)</td>
      <td>123</td>
      <td>oregano_rozemarijn</td>
    </tr>
    <tr>
      <th>201</th>
      <td>(bakmeel, basterdsuiker)</td>
      <td>122</td>
      <td>bakmeel_basterdsuiker</td>
    </tr>
    <tr>
      <th>202</th>
      <td>(tia, maria)</td>
      <td>122</td>
      <td>tia_maria</td>
    </tr>
    <tr>
      <th>203</th>
      <td>(john, west)</td>
      <td>120</td>
      <td>john_west</td>
    </tr>
    <tr>
      <th>204</th>
      <td>(donker, bier)</td>
      <td>120</td>
      <td>donker_bier</td>
    </tr>
    <tr>
      <th>205</th>
      <td>(danish, blue)</td>
      <td>120</td>
      <td>danish_blue</td>
    </tr>
    <tr>
      <th>206</th>
      <td>(blue, band)</td>
      <td>118</td>
      <td>blue_band</td>
    </tr>
    <tr>
      <th>207</th>
      <td>(poedersuiker, bestrooien)</td>
      <td>117</td>
      <td>poedersuiker_bestrooien</td>
    </tr>
    <tr>
      <th>208</th>
      <td>(roomboter, bladerdeeg)</td>
      <td>117</td>
      <td>roomboter_bladerdeeg</td>
    </tr>
    <tr>
      <th>209</th>
      <td>(wortel, bleekselderij)</td>
      <td>117</td>
      <td>wortel_bleekselderij</td>
    </tr>
    <tr>
      <th>210</th>
      <td>(geroosterde, sesamzaadjes)</td>
      <td>117</td>
      <td>geroosterde_sesamzaadjes</td>
    </tr>
    <tr>
      <th>211</th>
      <td>(sour, cream)</td>
      <td>117</td>
      <td>sour_cream</td>
    </tr>
    <tr>
      <th>212</th>
      <td>(ketoembar, djinten)</td>
      <td>117</td>
      <td>ketoembar_djinten</td>
    </tr>
    <tr>
      <th>213</th>
      <td>(bolletjes, gember)</td>
      <td>116</td>
      <td>bolletjes_gember</td>
    </tr>
    <tr>
      <th>214</th>
      <td>(italiaanse, kruidenmix)</td>
      <td>116</td>
      <td>italiaanse_kruidenmix</td>
    </tr>
    <tr>
      <th>215</th>
      <td>(poedersuiker, vanillesuiker)</td>
      <td>116</td>
      <td>poedersuiker_vanillesuiker</td>
    </tr>
    <tr>
      <th>216</th>
      <td>(hard, gekookt)</td>
      <td>116</td>
      <td>hard_gekookt</td>
    </tr>
    <tr>
      <th>217</th>
      <td>(pruimen, pit)</td>
      <td>116</td>
      <td>pruimen_pit</td>
    </tr>
    <tr>
      <th>218</th>
      <td>(aardbeien, frambozen)</td>
      <td>116</td>
      <td>aardbeien_frambozen</td>
    </tr>
    <tr>
      <th>219</th>
      <td>(à, c)</td>
      <td>116</td>
      <td>à_c</td>
    </tr>
    <tr>
      <th>220</th>
      <td>(tomato, frito)</td>
      <td>115</td>
      <td>tomato_frito</td>
    </tr>
    <tr>
      <th>221</th>
      <td>(rozemarijn, oregano)</td>
      <td>115</td>
      <td>rozemarijn_oregano</td>
    </tr>
    <tr>
      <th>222</th>
      <td>(laurierblaadjes, kruidnagels)</td>
      <td>115</td>
      <td>laurierblaadjes_kruidnagels</td>
    </tr>
    <tr>
      <th>223</th>
      <td>(tonijn, uitgelekt)</td>
      <td>115</td>
      <td>tonijn_uitgelekt</td>
    </tr>
    <tr>
      <th>224</th>
      <td>(roomboter, kamertemperatuur)</td>
      <td>114</td>
      <td>roomboter_kamertemperatuur</td>
    </tr>
    <tr>
      <th>225</th>
      <td>(rood, geel)</td>
      <td>114</td>
      <td>rood_geel</td>
    </tr>
    <tr>
      <th>226</th>
      <td>(santa, maria)</td>
      <td>114</td>
      <td>santa_maria</td>
    </tr>
    <tr>
      <th>227</th>
      <td>(kerrie, djawa)</td>
      <td>114</td>
      <td>kerrie_djawa</td>
    </tr>
    <tr>
      <th>228</th>
      <td>(maggi, basis)</td>
      <td>113</td>
      <td>maggi_basis</td>
    </tr>
    <tr>
      <th>229</th>
      <td>(blue, curacao)</td>
      <td>113</td>
      <td>blue_curacao</td>
    </tr>
    <tr>
      <th>230</th>
      <td>(eiwitten, poedersuiker)</td>
      <td>112</td>
      <td>eiwitten_poedersuiker</td>
    </tr>
    <tr>
      <th>231</th>
      <td>(draadjes, saffraan)</td>
      <td>112</td>
      <td>draadjes_saffraan</td>
    </tr>
    <tr>
      <th>232</th>
      <td>(gin, vruchtensap)</td>
      <td>111</td>
      <td>gin_vruchtensap</td>
    </tr>
    <tr>
      <th>233</th>
      <td>(rasp, sap)</td>
      <td>111</td>
      <td>rasp_sap</td>
    </tr>
    <tr>
      <th>234</th>
      <td>(molen, smaak)</td>
      <td>110</td>
      <td>molen_smaak</td>
    </tr>
    <tr>
      <th>235</th>
      <td>(pure, chocola)</td>
      <td>110</td>
      <td>pure_chocola</td>
    </tr>
    <tr>
      <th>236</th>
      <td>(turks, brood)</td>
      <td>109</td>
      <td>turks_brood</td>
    </tr>
    <tr>
      <th>237</th>
      <td>(casa, fiesta)</td>
      <td>109</td>
      <td>casa_fiesta</td>
    </tr>
    <tr>
      <th>238</th>
      <td>(rijpe, peren)</td>
      <td>109</td>
      <td>rijpe_peren</td>
    </tr>
    <tr>
      <th>239</th>
      <td>(fijne, kruiden)</td>
      <td>108</td>
      <td>fijne_kruiden</td>
    </tr>
    <tr>
      <th>240</th>
      <td>(mayonaise, zure)</td>
      <td>108</td>
      <td>mayonaise_zure</td>
    </tr>
    <tr>
      <th>241</th>
      <td>(rundergehakt, paneermeel)</td>
      <td>108</td>
      <td>rundergehakt_paneermeel</td>
    </tr>
    <tr>
      <th>242</th>
      <td>(belegen, geraspt)</td>
      <td>108</td>
      <td>belegen_geraspt</td>
    </tr>
    <tr>
      <th>243</th>
      <td>(tutti, frutti)</td>
      <td>107</td>
      <td>tutti_frutti</td>
    </tr>
    <tr>
      <th>244</th>
      <td>(gemberwortel, geraspt)</td>
      <td>107</td>
      <td>gemberwortel_geraspt</td>
    </tr>
    <tr>
      <th>245</th>
      <td>(pijnboompitten, geroosterd)</td>
      <td>107</td>
      <td>pijnboompitten_geroosterd</td>
    </tr>
    <tr>
      <th>246</th>
      <td>(vissaus, nam)</td>
      <td>106</td>
      <td>vissaus_nam</td>
    </tr>
    <tr>
      <th>247</th>
      <td>(kersen, pit)</td>
      <td>106</td>
      <td>kersen_pit</td>
    </tr>
    <tr>
      <th>248</th>
      <td>(sereh, citroengras)</td>
      <td>105</td>
      <td>sereh_citroengras</td>
    </tr>
    <tr>
      <th>249</th>
      <td>(rood, groen)</td>
      <td>105</td>
      <td>rood_groen</td>
    </tr>
    <tr>
      <th>250</th>
      <td>(appel, geschild)</td>
      <td>105</td>
      <td>appel_geschild</td>
    </tr>
    <tr>
      <th>251</th>
      <td>(stronkjes, witloof)</td>
      <td>105</td>
      <td>stronkjes_witloof</td>
    </tr>
    <tr>
      <th>252</th>
      <td>(dorange, vruchtensap)</td>
      <td>105</td>
      <td>dorange_vruchtensap</td>
    </tr>
    <tr>
      <th>253</th>
      <td>(faja, lobi)</td>
      <td>105</td>
      <td>faja_lobi</td>
    </tr>
    <tr>
      <th>254</th>
      <td>(verschillende, soorten)</td>
      <td>104</td>
      <td>verschillende_soorten</td>
    </tr>
    <tr>
      <th>255</th>
      <td>(gist, lauwe)</td>
      <td>104</td>
      <td>gist_lauwe</td>
    </tr>
    <tr>
      <th>256</th>
      <td>(zure, mayonaise)</td>
      <td>104</td>
      <td>zure_mayonaise</td>
    </tr>
    <tr>
      <th>257</th>
      <td>(groente, bouillon)</td>
      <td>104</td>
      <td>groente_bouillon</td>
    </tr>
    <tr>
      <th>258</th>
      <td>(meng, ingrediënten)</td>
      <td>103</td>
      <td>meng_ingrediënten</td>
    </tr>
    <tr>
      <th>259</th>
      <td>(mayonaise, tomatenketchup)</td>
      <td>103</td>
      <td>mayonaise_tomatenketchup</td>
    </tr>
    <tr>
      <th>260</th>
      <td>(gekookte, bietjes)</td>
      <td>102</td>
      <td>gekookte_bietjes</td>
    </tr>
    <tr>
      <th>261</th>
      <td>(mager, rundvlees)</td>
      <td>102</td>
      <td>mager_rundvlees</td>
    </tr>
    <tr>
      <th>262</th>
      <td>(geroosterd, brood)</td>
      <td>102</td>
      <td>geroosterd_brood</td>
    </tr>
    <tr>
      <th>263</th>
      <td>(mayonaise, ketchup)</td>
      <td>101</td>
      <td>mayonaise_ketchup</td>
    </tr>
    <tr>
      <th>264</th>
      <td>(bakken, braden)</td>
      <td>101</td>
      <td>bakken_braden</td>
    </tr>
    <tr>
      <th>265</th>
      <td>(sap, limoenen)</td>
      <td>101</td>
      <td>sap_limoenen</td>
    </tr>
    <tr>
      <th>266</th>
      <td>(bolletjes, mozzarella)</td>
      <td>101</td>
      <td>bolletjes_mozzarella</td>
    </tr>
    <tr>
      <th>267</th>
      <td>(gefruite, uitjes)</td>
      <td>101</td>
      <td>gefruite_uitjes</td>
    </tr>
    <tr>
      <th>268</th>
      <td>(aceto, balsamico)</td>
      <td>101</td>
      <td>aceto_balsamico</td>
    </tr>
    <tr>
      <th>269</th>
      <td>(spinazie, creme)</td>
      <td>101</td>
      <td>spinazie_creme</td>
    </tr>
    <tr>
      <th>270</th>
      <td>(wit, bruin)</td>
      <td>100</td>
      <td>wit_bruin</td>
    </tr>
    <tr>
      <th>271</th>
      <td>(shii, takes)</td>
      <td>100</td>
      <td>shii_takes</td>
    </tr>
    <tr>
      <th>272</th>
      <td>(vet, spek)</td>
      <td>100</td>
      <td>vet_spek</td>
    </tr>
    <tr>
      <th>273</th>
      <td>(bruin, bier)</td>
      <td>100</td>
      <td>bruin_bier</td>
    </tr>
    <tr>
      <th>274</th>
      <td>(gekneusde, peperkorrels)</td>
      <td>100</td>
      <td>gekneusde_peperkorrels</td>
    </tr>
    <tr>
      <th>275</th>
      <td>(gin, vermouth)</td>
      <td>100</td>
      <td>gin_vermouth</td>
    </tr>
    <tr>
      <th>276</th>
      <td>(roomkaas, naturel)</td>
      <td>100</td>
      <td>roomkaas_naturel</td>
    </tr>
    <tr>
      <th>277</th>
      <td>(feta, verkruimeld)</td>
      <td>100</td>
      <td>feta_verkruimeld</td>
    </tr>
    <tr>
      <th>278</th>
      <td>(curry, pasta)</td>
      <td>99</td>
      <td>curry_pasta</td>
    </tr>
    <tr>
      <th>279</th>
      <td>(zachte, margarine)</td>
      <td>99</td>
      <td>zachte_margarine</td>
    </tr>
    <tr>
      <th>280</th>
      <td>(pasta, penne)</td>
      <td>99</td>
      <td>pasta_penne</td>
    </tr>
    <tr>
      <th>281</th>
      <td>(jonge, spinazie)</td>
      <td>98</td>
      <td>jonge_spinazie</td>
    </tr>
    <tr>
      <th>282</th>
      <td>(koude, roomboter)</td>
      <td>98</td>
      <td>koude_roomboter</td>
    </tr>
    <tr>
      <th>283</th>
      <td>(tonijn, naturel)</td>
      <td>98</td>
      <td>tonijn_naturel</td>
    </tr>
    <tr>
      <th>284</th>
      <td>(risotto, rijst)</td>
      <td>98</td>
      <td>risotto_rijst</td>
    </tr>
    <tr>
      <th>285</th>
      <td>(komkommer, geschild)</td>
      <td>98</td>
      <td>komkommer_geschild</td>
    </tr>
    <tr>
      <th>286</th>
      <td>(afgespoeld, uitgelekt)</td>
      <td>98</td>
      <td>afgespoeld_uitgelekt</td>
    </tr>
    <tr>
      <th>287</th>
      <td>(penne, rigate)</td>
      <td>98</td>
      <td>penne_rigate</td>
    </tr>
    <tr>
      <th>288</th>
      <td>(grof, grof)</td>
      <td>97</td>
      <td>grof_grof</td>
    </tr>
    <tr>
      <th>289</th>
      <td>(rood, fruit)</td>
      <td>97</td>
      <td>rood_fruit</td>
    </tr>
    <tr>
      <th>290</th>
      <td>(smaak, toevoegen)</td>
      <td>96</td>
      <td>smaak_toevoegen</td>
    </tr>
    <tr>
      <th>291</th>
      <td>(fleur, sel)</td>
      <td>96</td>
      <td>fleur_sel</td>
    </tr>
    <tr>
      <th>292</th>
      <td>(gesnipperde, sjalot)</td>
      <td>95</td>
      <td>gesnipperde_sjalot</td>
    </tr>
    <tr>
      <th>293</th>
      <td>(chocolade, puur)</td>
      <td>95</td>
      <td>chocolade_puur</td>
    </tr>
    <tr>
      <th>294</th>
      <td>(geknipt, c)</td>
      <td>95</td>
      <td>geknipt_c</td>
    </tr>
    <tr>
      <th>295</th>
      <td>(rozemarijn, salie)</td>
      <td>94</td>
      <td>rozemarijn_salie</td>
    </tr>
    <tr>
      <th>296</th>
      <td>(knoflookpoeder, uienpoeder)</td>
      <td>94</td>
      <td>knoflookpoeder_uienpoeder</td>
    </tr>
    <tr>
      <th>297</th>
      <td>(goed, uitgelekt)</td>
      <td>94</td>
      <td>goed_uitgelekt</td>
    </tr>
    <tr>
      <th>298</th>
      <td>(zaadjes, ontdaan)</td>
      <td>94</td>
      <td>zaadjes_ontdaan</td>
    </tr>
    <tr>
      <th>299</th>
      <td>(basterdsuiker, zelfrijzend)</td>
      <td>94</td>
      <td>basterdsuiker_zelfrijzend</td>
    </tr>
    <tr>
      <th>300</th>
      <td>(appels, elstar)</td>
      <td>93</td>
      <td>appels_elstar</td>
    </tr>
    <tr>
      <th>301</th>
      <td>(kurkuma, geelwortel)</td>
      <td>93</td>
      <td>kurkuma_geelwortel</td>
    </tr>
    <tr>
      <th>302</th>
      <td>(olijven, kappertjes)</td>
      <td>93</td>
      <td>olijven_kappertjes</td>
    </tr>
    <tr>
      <th>303</th>
      <td>(tante, fanny)</td>
      <td>93</td>
      <td>tante_fanny</td>
    </tr>
    <tr>
      <th>304</th>
      <td>(lichte, chinese)</td>
      <td>92</td>
      <td>lichte_chinese</td>
    </tr>
    <tr>
      <th>305</th>
      <td>(zeezout, smaak)</td>
      <td>92</td>
      <td>zeezout_smaak</td>
    </tr>
    <tr>
      <th>306</th>
      <td>(kappertjes, uitgelekt)</td>
      <td>92</td>
      <td>kappertjes_uitgelekt</td>
    </tr>
    <tr>
      <th>307</th>
      <td>(preien, ringen)</td>
      <td>92</td>
      <td>preien_ringen</td>
    </tr>
    <tr>
      <th>308</th>
      <td>(amandelen, poedersuiker)</td>
      <td>92</td>
      <td>amandelen_poedersuiker</td>
    </tr>
    <tr>
      <th>309</th>
      <td>(mexicaanse, kruiden)</td>
      <td>92</td>
      <td>mexicaanse_kruiden</td>
    </tr>
    <tr>
      <th>310</th>
      <td>(sjalotje, gepeld)</td>
      <td>91</td>
      <td>sjalotje_gepeld</td>
    </tr>
    <tr>
      <th>311</th>
      <td>(spaanse, pepertjes)</td>
      <td>91</td>
      <td>spaanse_pepertjes</td>
    </tr>
    <tr>
      <th>312</th>
      <td>(voorgekookte, krieltjes)</td>
      <td>91</td>
      <td>voorgekookte_krieltjes</td>
    </tr>
    <tr>
      <th>313</th>
      <td>(mango, chutney)</td>
      <td>91</td>
      <td>mango_chutney</td>
    </tr>
    <tr>
      <th>314</th>
      <td>(kerstomaatjes, gehalveerd)</td>
      <td>91</td>
      <td>kerstomaatjes_gehalveerd</td>
    </tr>
    <tr>
      <th>315</th>
      <td>(cottage, cheese)</td>
      <td>90</td>
      <td>cottage_cheese</td>
    </tr>
    <tr>
      <th>316</th>
      <td>(oregano, gedroogd)</td>
      <td>90</td>
      <td>oregano_gedroogd</td>
    </tr>
    <tr>
      <th>317</th>
      <td>(h, h)</td>
      <td>90</td>
      <td>h_h</td>
    </tr>
    <tr>
      <th>318</th>
      <td>(kruiden, dille)</td>
      <td>90</td>
      <td>kruiden_dille</td>
    </tr>
    <tr>
      <th>319</th>
      <td>(noilly, prat)</td>
      <td>90</td>
      <td>noilly_prat</td>
    </tr>
    <tr>
      <th>320</th>
      <td>(bolletjes, vanille)</td>
      <td>89</td>
      <td>bolletjes_vanille</td>
    </tr>
    <tr>
      <th>321</th>
      <td>(chinese, rijstwijn)</td>
      <td>89</td>
      <td>chinese_rijstwijn</td>
    </tr>
    <tr>
      <th>322</th>
      <td>(lichte, basterdsuiker)</td>
      <td>89</td>
      <td>lichte_basterdsuiker</td>
    </tr>
    <tr>
      <th>323</th>
      <td>(turkse, yoghurt)</td>
      <td>89</td>
      <td>turkse_yoghurt</td>
    </tr>
    <tr>
      <th>324</th>
      <td>(erg, lekker)</td>
      <td>89</td>
      <td>erg_lekker</td>
    </tr>
    <tr>
      <th>325</th>
      <td>(hot, curry)</td>
      <td>88</td>
      <td>hot_curry</td>
    </tr>
    <tr>
      <th>326</th>
      <td>(mozzarella, parmezaanse)</td>
      <td>88</td>
      <td>mozzarella_parmezaanse</td>
    </tr>
    <tr>
      <th>327</th>
      <td>(roomboter, margarine)</td>
      <td>88</td>
      <td>roomboter_margarine</td>
    </tr>
    <tr>
      <th>328</th>
      <td>(cayennepeper, smaak)</td>
      <td>87</td>
      <td>cayennepeper_smaak</td>
    </tr>
    <tr>
      <th>329</th>
      <td>(gekookte, bieten)</td>
      <td>87</td>
      <td>gekookte_bieten</td>
    </tr>
    <tr>
      <th>330</th>
      <td>(baking, soda)</td>
      <td>87</td>
      <td>baking_soda</td>
    </tr>
    <tr>
      <th>331</th>
      <td>(solo, vloeibaar)</td>
      <td>87</td>
      <td>solo_vloeibaar</td>
    </tr>
    <tr>
      <th>332</th>
      <td>(rasp, sinaasappel)</td>
      <td>87</td>
      <td>rasp_sinaasappel</td>
    </tr>
    <tr>
      <th>333</th>
      <td>(kappertjes, olijven)</td>
      <td>87</td>
      <td>kappertjes_olijven</td>
    </tr>
    <tr>
      <th>334</th>
      <td>(struik, paksoi)</td>
      <td>87</td>
      <td>struik_paksoi</td>
    </tr>
    <tr>
      <th>335</th>
      <td>(creme, cassis)</td>
      <td>86</td>
      <td>creme_cassis</td>
    </tr>
    <tr>
      <th>336</th>
      <td>(worcestershire, saus)</td>
      <td>86</td>
      <td>worcestershire_saus</td>
    </tr>
    <tr>
      <th>337</th>
      <td>(tomaat, komkommer)</td>
      <td>86</td>
      <td>tomaat_komkommer</td>
    </tr>
    <tr>
      <th>338</th>
      <td>(kippenbouillon, opgelost)</td>
      <td>86</td>
      <td>kippenbouillon_opgelost</td>
    </tr>
    <tr>
      <th>339</th>
      <td>(kippen, bouillon)</td>
      <td>86</td>
      <td>kippen_bouillon</td>
    </tr>
    <tr>
      <th>340</th>
      <td>(moten, zalm)</td>
      <td>85</td>
      <td>moten_zalm</td>
    </tr>
    <tr>
      <th>341</th>
      <td>(zoete, aardappels)</td>
      <td>85</td>
      <td>zoete_aardappels</td>
    </tr>
    <tr>
      <th>342</th>
      <td>(asperges, asperges)</td>
      <td>85</td>
      <td>asperges_asperges</td>
    </tr>
    <tr>
      <th>343</th>
      <td>(kervel, dragon)</td>
      <td>85</td>
      <td>kervel_dragon</td>
    </tr>
    <tr>
      <th>344</th>
      <td>(milde, kerriepoeder)</td>
      <td>85</td>
      <td>milde_kerriepoeder</td>
    </tr>
    <tr>
      <th>345</th>
      <td>(i, v)</td>
      <td>85</td>
      <td>i_v</td>
    </tr>
    <tr>
      <th>346</th>
      <td>(tan, original)</td>
      <td>85</td>
      <td>tan_original</td>
    </tr>
    <tr>
      <th>347</th>
      <td>(geroosterde, pinda)</td>
      <td>84</td>
      <td>geroosterde_pinda</td>
    </tr>
    <tr>
      <th>348</th>
      <td>(hollandse, geitenkaas)</td>
      <td>83</td>
      <td>hollandse_geitenkaas</td>
    </tr>
    <tr>
      <th>349</th>
      <td>(droogkokende, rijst)</td>
      <td>83</td>
      <td>droogkokende_rijst</td>
    </tr>
    <tr>
      <th>350</th>
      <td>(medium, sherry)</td>
      <td>83</td>
      <td>medium_sherry</td>
    </tr>
    <tr>
      <th>351</th>
      <td>(zoute, haringen)</td>
      <td>83</td>
      <td>zoute_haringen</td>
    </tr>
    <tr>
      <th>352</th>
      <td>(lente, ringetjes)</td>
      <td>83</td>
      <td>lente_ringetjes</td>
    </tr>
    <tr>
      <th>353</th>
      <td>(vanillesuiker, bakpoeder)</td>
      <td>83</td>
      <td>vanillesuiker_bakpoeder</td>
    </tr>
    <tr>
      <th>354</th>
      <td>(mayonaise, creme)</td>
      <td>83</td>
      <td>mayonaise_creme</td>
    </tr>
    <tr>
      <th>355</th>
      <td>(gesmolten, roomboter)</td>
      <td>83</td>
      <td>gesmolten_roomboter</td>
    </tr>
    <tr>
      <th>356</th>
      <td>(sjalotjes, gepeld)</td>
      <td>83</td>
      <td>sjalotjes_gepeld</td>
    </tr>
    <tr>
      <th>357</th>
      <td>(dressing, mayonaise)</td>
      <td>82</td>
      <td>dressing_mayonaise</td>
    </tr>
    <tr>
      <th>358</th>
      <td>(bonen, uitgelekt)</td>
      <td>82</td>
      <td>bonen_uitgelekt</td>
    </tr>
    <tr>
      <th>359</th>
      <td>(djeroek, poeroet)</td>
      <td>82</td>
      <td>djeroek_poeroet</td>
    </tr>
    <tr>
      <th>360</th>
      <td>(corned, beef)</td>
      <td>82</td>
      <td>corned_beef</td>
    </tr>
    <tr>
      <th>361</th>
      <td>(kookroom, light)</td>
      <td>81</td>
      <td>kookroom_light</td>
    </tr>
    <tr>
      <th>362</th>
      <td>(langkorrelige, rijst)</td>
      <td>81</td>
      <td>langkorrelige_rijst</td>
    </tr>
    <tr>
      <th>363</th>
      <td>(worcestershire, sauce)</td>
      <td>81</td>
      <td>worcestershire_sauce</td>
    </tr>
    <tr>
      <th>364</th>
      <td>(ananas, sap)</td>
      <td>81</td>
      <td>ananas_sap</td>
    </tr>
    <tr>
      <th>365</th>
      <td>(sherry, maizena)</td>
      <td>81</td>
      <td>sherry_maizena</td>
    </tr>
    <tr>
      <th>366</th>
      <td>(geroosterd, sesamzaad)</td>
      <td>81</td>
      <td>geroosterd_sesamzaad</td>
    </tr>
    <tr>
      <th>367</th>
      <td>(zachte, roomkaas)</td>
      <td>81</td>
      <td>zachte_roomkaas</td>
    </tr>
    <tr>
      <th>368</th>
      <td>(gezouten, cashewnoten)</td>
      <td>80</td>
      <td>gezouten_cashewnoten</td>
    </tr>
    <tr>
      <th>369</th>
      <td>(bakpoeder, vanillesuiker)</td>
      <td>80</td>
      <td>bakpoeder_vanillesuiker</td>
    </tr>
    <tr>
      <th>370</th>
      <td>(rozijnen, rum)</td>
      <td>80</td>
      <td>rozijnen_rum</td>
    </tr>
    <tr>
      <th>371</th>
      <td>(gezeefde, poedersuiker)</td>
      <td>80</td>
      <td>gezeefde_poedersuiker</td>
    </tr>
    <tr>
      <th>372</th>
      <td>(aardappel, geschild)</td>
      <td>80</td>
      <td>aardappel_geschild</td>
    </tr>
    <tr>
      <th>373</th>
      <td>(gewelde, abrikozen)</td>
      <td>80</td>
      <td>gewelde_abrikozen</td>
    </tr>
    <tr>
      <th>374</th>
      <td>(chinese, eiermie)</td>
      <td>80</td>
      <td>chinese_eiermie</td>
    </tr>
    <tr>
      <th>375</th>
      <td>(chili, saus)</td>
      <td>80</td>
      <td>chili_saus</td>
    </tr>
    <tr>
      <th>376</th>
      <td>(ontveld, ontpit)</td>
      <td>80</td>
      <td>ontveld_ontpit</td>
    </tr>
    <tr>
      <th>377</th>
      <td>(buffel, mozzarella)</td>
      <td>80</td>
      <td>buffel_mozzarella</td>
    </tr>
    <tr>
      <th>378</th>
      <td>(cheddar, geraspt)</td>
      <td>80</td>
      <td>cheddar_geraspt</td>
    </tr>
    <tr>
      <th>379</th>
      <td>(geroosterde, amandelen)</td>
      <td>79</td>
      <td>geroosterde_amandelen</td>
    </tr>
    <tr>
      <th>380</th>
      <td>(deeg, gist)</td>
      <td>79</td>
      <td>deeg_gist</td>
    </tr>
    <tr>
      <th>381</th>
      <td>(pindakaas, sambal)</td>
      <td>79</td>
      <td>pindakaas_sambal</td>
    </tr>
    <tr>
      <th>382</th>
      <td>(geschild, klokhuis)</td>
      <td>79</td>
      <td>geschild_klokhuis</td>
    </tr>
    <tr>
      <th>383</th>
      <td>(zoetzure, appels)</td>
      <td>79</td>
      <td>zoetzure_appels</td>
    </tr>
    <tr>
      <th>384</th>
      <td>(volle, kwark)</td>
      <td>79</td>
      <td>volle_kwark</td>
    </tr>
    <tr>
      <th>385</th>
      <td>(gele, courgette)</td>
      <td>78</td>
      <td>gele_courgette</td>
    </tr>
    <tr>
      <th>386</th>
      <td>(zonnebloemolie, c)</td>
      <td>78</td>
      <td>zonnebloemolie_c</td>
    </tr>
    <tr>
      <th>387</th>
      <td>(geschild, partjes)</td>
      <td>78</td>
      <td>geschild_partjes</td>
    </tr>
    <tr>
      <th>388</th>
      <td>(aardbeien, poedersuiker)</td>
      <td>78</td>
      <td>aardbeien_poedersuiker</td>
    </tr>
    <tr>
      <th>389</th>
      <td>(penne, pasta)</td>
      <td>78</td>
      <td>penne_pasta</td>
    </tr>
    <tr>
      <th>390</th>
      <td>(koffie, rum)</td>
      <td>77</td>
      <td>koffie_rum</td>
    </tr>
    <tr>
      <th>391</th>
      <td>(blanke, hazelnoten)</td>
      <td>77</td>
      <td>blanke_hazelnoten</td>
    </tr>
    <tr>
      <th>392</th>
      <td>(fraiche, mayonaise)</td>
      <td>77</td>
      <td>fraiche_mayonaise</td>
    </tr>
    <tr>
      <th>393</th>
      <td>(broodkruim, paneermeel)</td>
      <td>77</td>
      <td>broodkruim_paneermeel</td>
    </tr>
    <tr>
      <th>394</th>
      <td>(tabasco, smaak)</td>
      <td>77</td>
      <td>tabasco_smaak</td>
    </tr>
    <tr>
      <th>395</th>
      <td>(dressing, balsamico)</td>
      <td>77</td>
      <td>dressing_balsamico</td>
    </tr>
    <tr>
      <th>396</th>
      <td>(pitloze, druiven)</td>
      <td>77</td>
      <td>pitloze_druiven</td>
    </tr>
    <tr>
      <th>397</th>
      <td>(ansjovisfilets, kappertjes)</td>
      <td>77</td>
      <td>ansjovisfilets_kappertjes</td>
    </tr>
    <tr>
      <th>398</th>
      <td>(chocolade, chocolade)</td>
      <td>77</td>
      <td>chocolade_chocolade</td>
    </tr>
    <tr>
      <th>399</th>
      <td>(appel, peer)</td>
      <td>77</td>
      <td>appel_peer</td>
    </tr>
    <tr>
      <th>400</th>
      <td>(gekookte, gepelde)</td>
      <td>76</td>
      <td>gekookte_gepelde</td>
    </tr>
    <tr>
      <th>401</th>
      <td>(chocolade, poedersuiker)</td>
      <td>76</td>
      <td>chocolade_poedersuiker</td>
    </tr>
    <tr>
      <th>402</th>
      <td>(sla, tomaat)</td>
      <td>76</td>
      <td>sla_tomaat</td>
    </tr>
    <tr>
      <th>403</th>
      <td>(gewelde, rozijnen)</td>
      <td>76</td>
      <td>gewelde_rozijnen</td>
    </tr>
    <tr>
      <th>404</th>
      <td>(verschillende, kleuren)</td>
      <td>76</td>
      <td>verschillende_kleuren</td>
    </tr>
    <tr>
      <th>405</th>
      <td>(kersen, sap)</td>
      <td>76</td>
      <td>kersen_sap</td>
    </tr>
    <tr>
      <th>406</th>
      <td>(bettine, blanc)</td>
      <td>76</td>
      <td>bettine_blanc</td>
    </tr>
    <tr>
      <th>407</th>
      <td>(zoet, zure)</td>
      <td>75</td>
      <td>zoet_zure</td>
    </tr>
    <tr>
      <th>408</th>
      <td>(gist, lauw)</td>
      <td>75</td>
      <td>gist_lauw</td>
    </tr>
    <tr>
      <th>409</th>
      <td>(appel, granny)</td>
      <td>75</td>
      <td>appel_granny</td>
    </tr>
    <tr>
      <th>410</th>
      <td>(i, d)</td>
      <td>75</td>
      <td>i_d</td>
    </tr>
    <tr>
      <th>411</th>
      <td>(stronk, broccoli)</td>
      <td>75</td>
      <td>stronk_broccoli</td>
    </tr>
    <tr>
      <th>412</th>
      <td>(smalle, ringetjes)</td>
      <td>75</td>
      <td>smalle_ringetjes</td>
    </tr>
    <tr>
      <th>413</th>
      <td>(doorregen, runderlappen)</td>
      <td>75</td>
      <td>doorregen_runderlappen</td>
    </tr>
    <tr>
      <th>414</th>
      <td>(kruimig, kokende)</td>
      <td>74</td>
      <td>kruimig_kokende</td>
    </tr>
    <tr>
      <th>415</th>
      <td>(original, wok)</td>
      <td>74</td>
      <td>original_wok</td>
    </tr>
    <tr>
      <th>416</th>
      <td>(stelen, bleekselderij)</td>
      <td>74</td>
      <td>stelen_bleekselderij</td>
    </tr>
    <tr>
      <th>417</th>
      <td>(tarwebloem, bakpoeder)</td>
      <td>74</td>
      <td>tarwebloem_bakpoeder</td>
    </tr>
    <tr>
      <th>418</th>
      <td>(zaad, ontdaan)</td>
      <td>74</td>
      <td>zaad_ontdaan</td>
    </tr>
    <tr>
      <th>419</th>
      <td>(losgeklopt, paneermeel)</td>
      <td>74</td>
      <td>losgeklopt_paneermeel</td>
    </tr>
    <tr>
      <th>420</th>
      <td>(panklare, spinazie)</td>
      <td>74</td>
      <td>panklare_spinazie</td>
    </tr>
    <tr>
      <th>421</th>
      <td>(sambal, brandal)</td>
      <td>73</td>
      <td>sambal_brandal</td>
    </tr>
    <tr>
      <th>422</th>
      <td>(verkrijgbaar, toko)</td>
      <td>73</td>
      <td>verkrijgbaar_toko</td>
    </tr>
    <tr>
      <th>423</th>
      <td>(vergine, behoefte)</td>
      <td>73</td>
      <td>vergine_behoefte</td>
    </tr>
    <tr>
      <th>424</th>
      <td>(ringen, geperst)</td>
      <td>73</td>
      <td>ringen_geperst</td>
    </tr>
    <tr>
      <th>425</th>
      <td>(geel, groen)</td>
      <td>73</td>
      <td>geel_groen</td>
    </tr>
    <tr>
      <th>426</th>
      <td>(djinten, komijn)</td>
      <td>73</td>
      <td>djinten_komijn</td>
    </tr>
    <tr>
      <th>427</th>
      <td>(klont, roomboter)</td>
      <td>73</td>
      <td>klont_roomboter</td>
    </tr>
    <tr>
      <th>428</th>
      <td>(tomaat, ontveld)</td>
      <td>73</td>
      <td>tomaat_ontveld</td>
    </tr>
    <tr>
      <th>429</th>
      <td>(olijf, zonnebloemolie)</td>
      <td>73</td>
      <td>olijf_zonnebloemolie</td>
    </tr>
    <tr>
      <th>430</th>
      <td>(gula, djawa)</td>
      <td>72</td>
      <td>gula_djawa</td>
    </tr>
    <tr>
      <th>431</th>
      <td>(sinaasappel, sap)</td>
      <td>72</td>
      <td>sinaasappel_sap</td>
    </tr>
    <tr>
      <th>432</th>
      <td>(sla, keuze)</td>
      <td>72</td>
      <td>sla_keuze</td>
    </tr>
    <tr>
      <th>433</th>
      <td>(roze, zalm)</td>
      <td>72</td>
      <td>roze_zalm</td>
    </tr>
    <tr>
      <th>434</th>
      <td>(lichte, siroop)</td>
      <td>72</td>
      <td>lichte_siroop</td>
    </tr>
    <tr>
      <th>435</th>
      <td>(old, amsterdam)</td>
      <td>72</td>
      <td>old_amsterdam</td>
    </tr>
    <tr>
      <th>436</th>
      <td>(madam, jeanette)</td>
      <td>72</td>
      <td>madam_jeanette</td>
    </tr>
    <tr>
      <th>437</th>
      <td>(oud, wit)</td>
      <td>72</td>
      <td>oud_wit</td>
    </tr>
    <tr>
      <th>438</th>
      <td>(saus, mayonaise)</td>
      <td>71</td>
      <td>saus_mayonaise</td>
    </tr>
    <tr>
      <th>439</th>
      <td>(zaad, verwijderd)</td>
      <td>71</td>
      <td>zaad_verwijderd</td>
    </tr>
    <tr>
      <th>440</th>
      <td>(pit, kappertjes)</td>
      <td>71</td>
      <td>pit_kappertjes</td>
    </tr>
    <tr>
      <th>441</th>
      <td>(rasp, limoen)</td>
      <td>71</td>
      <td>rasp_limoen</td>
    </tr>
    <tr>
      <th>442</th>
      <td>(uienpoeder, knoflookpoeder)</td>
      <td>71</td>
      <td>uienpoeder_knoflookpoeder</td>
    </tr>
    <tr>
      <th>443</th>
      <td>(pijnboompitten, parmezaanse)</td>
      <td>71</td>
      <td>pijnboompitten_parmezaanse</td>
    </tr>
    <tr>
      <th>444</th>
      <td>(oil, vinegar)</td>
      <td>71</td>
      <td>oil_vinegar</td>
    </tr>
    <tr>
      <th>445</th>
      <td>(geroosterde, hazelnoten)</td>
      <td>70</td>
      <td>geroosterde_hazelnoten</td>
    </tr>
    <tr>
      <th>446</th>
      <td>(margarine, vetten)</td>
      <td>70</td>
      <td>margarine_vetten</td>
    </tr>
    <tr>
      <th>447</th>
      <td>(gepelde, grijze)</td>
      <td>70</td>
      <td>gepelde_grijze</td>
    </tr>
    <tr>
      <th>448</th>
      <td>(bosuitje, ringetjes)</td>
      <td>70</td>
      <td>bosuitje_ringetjes</td>
    </tr>
    <tr>
      <th>449</th>
      <td>(chili, poeder)</td>
      <td>70</td>
      <td>chili_poeder</td>
    </tr>
    <tr>
      <th>450</th>
      <td>(albert, heijn)</td>
      <td>70</td>
      <td>albert_heijn</td>
    </tr>
    <tr>
      <th>451</th>
      <td>(merg, vanillestokje)</td>
      <td>70</td>
      <td>merg_vanillestokje</td>
    </tr>
    <tr>
      <th>452</th>
      <td>(bakmeel, gezeefd)</td>
      <td>70</td>
      <td>bakmeel_gezeefd</td>
    </tr>
    <tr>
      <th>453</th>
      <td>(reep, pure)</td>
      <td>70</td>
      <td>reep_pure</td>
    </tr>
    <tr>
      <th>454</th>
      <td>(licht, geroosterd)</td>
      <td>69</td>
      <td>licht_geroosterd</td>
    </tr>
    <tr>
      <th>455</th>
      <td>(glazuur, poedersuiker)</td>
      <td>69</td>
      <td>glazuur_poedersuiker</td>
    </tr>
    <tr>
      <th>456</th>
      <td>(gezouten, roomboter)</td>
      <td>69</td>
      <td>gezouten_roomboter</td>
    </tr>
    <tr>
      <th>457</th>
      <td>(spek, dobbelsteentjes)</td>
      <td>69</td>
      <td>spek_dobbelsteentjes</td>
    </tr>
    <tr>
      <th>458</th>
      <td>(kaneelstokje, kruidnagels)</td>
      <td>69</td>
      <td>kaneelstokje_kruidnagels</td>
    </tr>
    <tr>
      <th>459</th>
      <td>(basterdsuiker, speculaaskruiden)</td>
      <td>69</td>
      <td>basterdsuiker_speculaaskruiden</td>
    </tr>
    <tr>
      <th>460</th>
      <td>(pikante, variant)</td>
      <td>69</td>
      <td>pikante_variant</td>
    </tr>
    <tr>
      <th>461</th>
      <td>(atjar, tjampoer)</td>
      <td>69</td>
      <td>atjar_tjampoer</td>
    </tr>
    <tr>
      <th>462</th>
      <td>(krieltjes, schil)</td>
      <td>69</td>
      <td>krieltjes_schil</td>
    </tr>
    <tr>
      <th>463</th>
      <td>(chocolade, cacao)</td>
      <td>68</td>
      <td>chocolade_cacao</td>
    </tr>
    <tr>
      <th>464</th>
      <td>(eierdooiers, eiwitten)</td>
      <td>68</td>
      <td>eierdooiers_eiwitten</td>
    </tr>
    <tr>
      <th>465</th>
      <td>(kruiden, rozemarijn)</td>
      <td>68</td>
      <td>kruiden_rozemarijn</td>
    </tr>
    <tr>
      <th>466</th>
      <td>(zoete, puntpaprika)</td>
      <td>68</td>
      <td>zoete_puntpaprika</td>
    </tr>
    <tr>
      <th>467</th>
      <td>(magere, ham)</td>
      <td>68</td>
      <td>magere_ham</td>
    </tr>
    <tr>
      <th>468</th>
      <td>(sambal, trassi)</td>
      <td>68</td>
      <td>sambal_trassi</td>
    </tr>
    <tr>
      <th>469</th>
      <td>(pepers, zaadjes)</td>
      <td>68</td>
      <td>pepers_zaadjes</td>
    </tr>
    <tr>
      <th>470</th>
      <td>(sugar, snaps)</td>
      <td>68</td>
      <td>sugar_snaps</td>
    </tr>
    <tr>
      <th>471</th>
      <td>(olijven, piment)</td>
      <td>67</td>
      <td>olijven_piment</td>
    </tr>
    <tr>
      <th>472</th>
      <td>(basterdsuiker, vanille)</td>
      <td>67</td>
      <td>basterdsuiker_vanille</td>
    </tr>
    <tr>
      <th>473</th>
      <td>(gember, siroop)</td>
      <td>67</td>
      <td>gember_siroop</td>
    </tr>
    <tr>
      <th>474</th>
      <td>(rijpe, banaan)</td>
      <td>67</td>
      <td>rijpe_banaan</td>
    </tr>
    <tr>
      <th>475</th>
      <td>(little, gem)</td>
      <td>67</td>
      <td>little_gem</td>
    </tr>
    <tr>
      <th>476</th>
      <td>(komkommer, tomaat)</td>
      <td>67</td>
      <td>komkommer_tomaat</td>
    </tr>
    <tr>
      <th>477</th>
      <td>(chinese, paddestoelen)</td>
      <td>67</td>
      <td>chinese_paddestoelen</td>
    </tr>
    <tr>
      <th>478</th>
      <td>(paneermeel, parmezaanse)</td>
      <td>67</td>
      <td>paneermeel_parmezaanse</td>
    </tr>
    <tr>
      <th>479</th>
      <td>(bosuitjes, ringen)</td>
      <td>67</td>
      <td>bosuitjes_ringen</td>
    </tr>
    <tr>
      <th>480</th>
      <td>(mee, garneren)</td>
      <td>67</td>
      <td>mee_garneren</td>
    </tr>
    <tr>
      <th>481</th>
      <td>(gezouten, pinda)</td>
      <td>67</td>
      <td>gezouten_pinda</td>
    </tr>
    <tr>
      <th>482</th>
      <td>(spinazie, gewassen)</td>
      <td>67</td>
      <td>spinazie_gewassen</td>
    </tr>
    <tr>
      <th>483</th>
      <td>(cognac, vieux)</td>
      <td>67</td>
      <td>cognac_vieux</td>
    </tr>
    <tr>
      <th>484</th>
      <td>(oelek, smaak)</td>
      <td>67</td>
      <td>oelek_smaak</td>
    </tr>
    <tr>
      <th>485</th>
      <td>(maggi, puree)</td>
      <td>67</td>
      <td>maggi_puree</td>
    </tr>
    <tr>
      <th>486</th>
      <td>(madame, jeanette)</td>
      <td>67</td>
      <td>madame_jeanette</td>
    </tr>
    <tr>
      <th>487</th>
      <td>(seasoning, mix)</td>
      <td>67</td>
      <td>seasoning_mix</td>
    </tr>
    <tr>
      <th>488</th>
      <td>(smalle, ringen)</td>
      <td>67</td>
      <td>smalle_ringen</td>
    </tr>
    <tr>
      <th>489</th>
      <td>(boursin, cuisine)</td>
      <td>67</td>
      <td>boursin_cuisine</td>
    </tr>
    <tr>
      <th>490</th>
      <td>(gemengde, groenten)</td>
      <td>66</td>
      <td>gemengde_groenten</td>
    </tr>
    <tr>
      <th>491</th>
      <td>(chorizo, worst)</td>
      <td>66</td>
      <td>chorizo_worst</td>
    </tr>
    <tr>
      <th>492</th>
      <td>(licht, geklopt)</td>
      <td>66</td>
      <td>licht_geklopt</td>
    </tr>
    <tr>
      <th>493</th>
      <td>(feta, olijven)</td>
      <td>66</td>
      <td>feta_olijven</td>
    </tr>
    <tr>
      <th>494</th>
      <td>(chili, con)</td>
      <td>66</td>
      <td>chili_con</td>
    </tr>
    <tr>
      <th>495</th>
      <td>(bbq, saus)</td>
      <td>66</td>
      <td>bbq_saus</td>
    </tr>
    <tr>
      <th>496</th>
      <td>(mager, ontbijtspek)</td>
      <td>66</td>
      <td>mager_ontbijtspek</td>
    </tr>
    <tr>
      <th>497</th>
      <td>(zure, augurken)</td>
      <td>66</td>
      <td>zure_augurken</td>
    </tr>
    <tr>
      <th>498</th>
      <td>(poedersuiker, vanille)</td>
      <td>65</td>
      <td>poedersuiker_vanille</td>
    </tr>
    <tr>
      <th>499</th>
      <td>(courgette, gele)</td>
      <td>65</td>
      <td>courgette_gele</td>
    </tr>
  </tbody>
</table>
</div>




```python
#FIND MOST COMMON TRIGRAMS
```


```python
def return_trigrams(words):
    trigrams = zip(words, words[1:], words[2:])
    return trigrams
```


```python
trigramvocabulary = Counter()
trigrams = rcp_c["ingrediënten"].str.split().apply(return_trigrams).apply(trigramvocabulary.update)
```


```python
trigram_freq = pd.DataFrame(trigramvocabulary.most_common(), columns = ['trigram', 'freq'])
```


```python
def concatenate_trigram(t):
    return str(t[0] + "_" + t[1] + "_" + t[2])
```


```python
trigram_freq["trigram_conc"] = trigram_freq.trigram.apply(concatenate_trigram)
```


```python
trigram_freq[0:5]
```




<div>
<style scoped>
    .dataframe tbody tr th:only-of-type {
        vertical-align: middle;
    }

    .dataframe tbody tr th {
        vertical-align: top;
    }

    .dataframe thead th {
        text-align: right;
    }
</style>
<table border="1" class="dataframe">
  <thead>
    <tr style="text-align: right;">
      <th></th>
      <th>trigram</th>
      <th>freq</th>
      <th>trigram_conc</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th>0</th>
      <td>(traditioneel, c, selectie)</td>
      <td>226</td>
      <td>traditioneel_c_selectie</td>
    </tr>
    <tr>
      <th>1</th>
      <td>(medium, dry, sherry)</td>
      <td>143</td>
      <td>medium_dry_sherry</td>
    </tr>
    <tr>
      <th>2</th>
      <td>(zelfrijzend, bakmeel, bakpoeder)</td>
      <td>139</td>
      <td>zelfrijzend_bakmeel_bakpoeder</td>
    </tr>
    <tr>
      <th>3</th>
      <td>(lente, uitjes, ringetjes)</td>
      <td>135</td>
      <td>lente_uitjes_ringetjes</td>
    </tr>
    <tr>
      <th>4</th>
      <td>(deeg, hartige, taart)</td>
      <td>129</td>
      <td>deeg_hartige_taart</td>
    </tr>
  </tbody>
</table>
</div>




```python
# SIEVING N GRAMS
```


```python
len(monogram_freq), len(bigram_freq), len(trigram_freq)
```




    (49902, 532479, 780147)




```python
monogram_freq.to_csv("monograms.csv", index = True)
```


```python
bigram_freq.to_csv("bigrams.csv", index = True)
```


```python
trigram_freq.to_csv("trigrams.csv", index = True)
```

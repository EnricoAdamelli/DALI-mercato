### Enrico Adamelli 
# Progetto DALI: mercato

The idea of this project is to implement the interaction between buyers and sellers in a market; 
Moreover: 
1) Every buyer has the following traits: name, age,  expendable amount of money. 
2) Every seller has the following traits: name, products to sell, total gain.
3) The market acts like a broker which notifies the sellers when a buyer is available to buy something and then allows every seller to communicate with each buyer.
# Class Diagram 
![Class Diagram](https://github.com/EnricoAdamelli/DALI-mercato/assets/64257821/fde5736e-d8d8-4b86-9cce-9c0e8588bd59)

<li> Buyer: For instance we have here 2 buyers, buyer1 and buyer2. Each buyer has different age, name and available money. In particular, buyer2 has age 15, therefore he won't be able to buy anything in the market. Each buyer, for instance buyer1, can communicate with a seller when the buyer chooses to buy a product of that particular seller and pays him. </li>
<li> Seller: For instance we have here 2 sellers, seller1 and seller2. Each seller has different name, income and products to sell. </li>
<li> Mercato: This class has to keep trace of all the buyers and sellers in the system. Moreover it keeps track of which buyer is communicating with which seller by using the matching.</li>

# Sequence Diagram
![Sequence1](https://github.com/EnricoAdamelli/DALI-mercato/assets/64257821/d55e92df-680d-4562-ba22-6b092625157a)

The first sequence diagram shows the start of the trading process. Fist of all, the user can choose which (seller) agent to activate asynchronously first. Evry agent (seller or buyer) present itself when they become active and register it as a past event. When both the buyer and the sellers are available (they are in the market which has asserted their type), the buyer can start the trading and the market notifies all the sellers that an adult buyer is available to purchase something. 

![Sequence2](https://github.com/EnricoAdamelli/DALI-mercato/assets/64257821/3cef297e-f380-44fb-b2ea-cdceaf9f959b)

The sequence diagram follows from the SD1. One by one, every seller tries to sell the products to the buyer depending on the amount of money he has. The buyer chooses the products he wants and buys them by adding them to its own shopping list. The market keeps trace of every trade. Moreover the buyer has to pay every object, so its amount of money decrease by the cost of that object, and the seller gains the same corresponding amount.

# Code

In the following section there are some snippets of code.
### Avvia trade
This is the code of a buyer:

```
avvia_trade :- impegnato(no), buyer_in(true), spesa(ListaSpesa),\+nonempty(ListaSpesa).
avvia_tradeI:- saldo_agente(X), eta_agente(Y), messageA(mercato, send_message(trading(Me, X, Y),Me)),
				retract(impegnato(no)), assert(impegnato(si)).
```

Every buyer starts a trade autonomously when the following conditions are fullfilled: He is not busy with a seller (so that he can work with one seller at a time), he is in the market and his cart is currently empty. Therefore, the buyer sends a message to the market declaring his age and money and will be busy from now on.

This is the code of market:

```
tradingE(Buyer, Saldo, Eta) :- Eta < 18, write(Buyer),write(' sei minorenne!!!'), nl.
tradingE(Buyer, Saldo, Eta) :- Eta >= 18, write(Buyer),write(' vorrebbe acquistare qualcosa!!!'),nl,avvisa_sellerA(Buyer, Saldo).
avvisa_sellerA(Buyer,Saldo) :> findall(Agent, (seller(Agent),\+matching(Buyer,Agent)), Sellers), send_message_to_sellersA(Sellers, Buyer, Saldo).
```

The market receives the message from the buyer and checks his age: if he is an adult buyer, then all sellers will be notified of the availability of that buyer.  

This is the code of a seller:

```
buyer_disponibileE(Buyer, Saldo):> write(Buyer), write(' vuoi comprare qualcosa? Hai saldo '), write(Saldo),nl,offri_prodotto(Buyer, Saldo).
```


Every seller knows that buyer <i>Buyer</i> wants to buy something and that he can spend at most <i>Saldo</i> amount of money. With these informations, the seller can start offering products.

### offri prodotto

```
offri_prodotto(Buyer, Saldo):> format('Offro prodotti a ~w che costano al massimo ~w euro... ~n ', [Buyer, Saldo]),
							 invia_tutti_gli_oggetti(Buyer, Saldo).
	
invia_tutti_gli_oggetti(Buyer, Saldo) :-
    lista_oggetti(Lista),
    invia_lista_oggetti(Lista, Buyer, Saldo).

invia_lista_oggetti([], Buyer, Saldo):- write('fine lista prodotti'),nl ,messageA(Buyer, send_message(check_spesa,Me)).
invia_lista_oggetti([[Nome, Valore]|AltriOggetti], Buyer, Saldo) :-
	write([Nome,Valore]),
	(\+dif(Nome, null), \+dif(Valore, null)-> write('Non posso venderti nulla, mi spiace!');
	Valore > Saldo -> write('Purtroppo non hai abbastanza soldi per questo prodotto!'),nl, invia_lista_oggetti(AltriOggetti, Buyer, Saldo);
	Valore =< Saldo -> invia_oggetto([Nome, Valore], Buyer), invia_lista_oggetti(AltriOggetti, Buyer, Saldo)).

invia_oggetto(Oggetto, Buyer) :- write('Ho questo prodotto: '), write(Oggetto), nl,
	messageA(mercato, send_message(match(Buyer,Me),Me)),
	messageA(Buyer, send_message(ricevi_oggetto(Oggetto),Me)).
```
The seller who offers a product will take into consideration the Buyer and his money. Recursively, the seller will pick and then send one by one the items he can sell to that buyer. If the seller does not have any product to sell, he does nothing. When he has sent all the possible products, the seller sends a _check_spesa_ message to the buyer. 

Buyer code:

```
ricevi_oggettoE(Oggetto):> spesa(ListaSpesa), append(ListaSpesa, [Oggetto], ListaCorrente),
			retractall(spesa(_)), assert(spesa(ListaCorrente)).
check_spesaE :> stampa_spesa, assert(check_spesa(complete)).

scegli_prodotto:- spesa(ListaSpesa) ,nonempty(ListaSpesa), check_spesa(complete).
scegli_prodottoI:> write('Scelgo un prodotto tra quelli disponibili....'),nl,
		spesa(ListaSpesa),
		acquisti(ListaAcquisti),
    		random_member(Oggetto, ListaSpesa),
	    	write('Voglio acquistare questo oggetto: '), write(Oggetto), nl,
		append(ListaAcquisti, [Oggetto], ListaAcquistiNew),
		retractall(acquisti(_)),
		assert(acquisti(ListaAcquistiNew)),
		delete(ListaSpesa, Oggetto, NuovaListaSpesa),
		retractall(spesa(ListaSpesa)),
	    	assert(spesa(NuovaListaSpesa)),
		acquistaProdottoA(Oggetto).
```

The buyer receives all the products sent by the seller but proceeds with the choice of the object just after the check_spesa event triggered. After choosing the object, he then pays the seller and adds it to his shopping list. 

### acquista prodotto
Buyer code: 

```
acquistaProdottoA([Nome,Costo]) :>
	spesa(ListaSpesa),
    	saldo_agente(Saldo),
    	Costo =< Saldo, 
    	NuovoSaldo is Saldo - Costo,
	retract(saldo_agente(Saldo)),
    	asserta(saldo_agente(NuovoSaldo)),
	write('Mi restano '),
	saldo_agente(X),
	write(X), write(' euro.'),nl,	
	messageA(mercato, send_message(acquisto_effettuato([Nome,Costo],Me),Me)),
	check_acquisti.
```

Market code:
```
acquisto_effettuatoE([Nome,Costo], Buyer):> check_match([Nome,Costo],Buyer).

check_match([Nome,Costo],Buyer):- matching(Buyer,Seller), write('Acquisto effettuato correttamente: '), write(Nome), write(' da '),write(Buyer), nl, 
								messageA(Seller, send_message(acquisto_done([Nome,Costo]),Me)).
check_match([Nome,Costo],Buyer):- \+matching(Buyer,Seller), write('I due agenti non stavano trattando!'),nl.

```

Seller code:  
```
acquisto_doneE([Nome,Costo]):> lista_oggetti(Lista),
    	delete(Lista, [Nome, Costo], NuovaLista),
    	retractall(lista_oggetti(_)),
    	assert(lista_oggetti(NuovaLista)),
 	write('Oggetto venduto: '), write([Nome, Costo]), nl,
	incassa(Costo).
```
The buyer purchases the object considering its cost _Costo_ and subtracts that amount from its money _Saldo_. In the end the amount of money of the buyer is equal to _Saldo - Costo = NuovoSaldo_. The market is notified of the action and check that everything is correct while the seller gains the cost of the sold product. Moreover, check_acquisti is used to check if the Buyer can still buy other products from the same seller.

# Execution example
**In this example, the input is entered in the following way:**
![sicstus input](https://github.com/EnricoAdamelli/DALI-mercato/assets/64257821/60593d09-4942-42af-93eb-244473514f7e)


$${\color{blue}Seller1 \space (and \space seller2) \space are \space started:}$$

![seller1Part1](https://github.com/EnricoAdamelli/DALI-mercato/assets/64257821/f2f8e273-f38c-4479-b983-2af225e9bd99)

$${\color{blue}The \space market \space assert \space them \space as \space sellers: }$$

![market_pt1](https://github.com/EnricoAdamelli/DALI-mercato/assets/64257821/2dbc4563-8157-40c7-b809-b1a3055b46b3)

$${\color{blue}buyer1 \space enters \space and \space is \space asserted \space as \space a \space buyer; \space Then \space it \space starts \space the \space trading:}$$

![buyer1](https://github.com/EnricoAdamelli/DALI-mercato/assets/64257821/2adb627c-9089-43d7-891e-fce81ac5836f)

$${\color{blue}Market \space output:}$$

![mercato_pt2](https://github.com/EnricoAdamelli/DALI-mercato/assets/64257821/398c023c-18bc-42ab-95a4-8a4ef2eaf4ec)

$${\color{blue}Seller1 \space output:}$$

![seller1](https://github.com/EnricoAdamelli/DALI-mercato/assets/64257821/b9cc6c27-a705-4e29-b0a7-bcbc86461973)

$${\color{blue}Seller2 \space output:}$$

![seller2](https://github.com/EnricoAdamelli/DALI-mercato/assets/64257821/af346bdd-2932-434a-b0b8-aedda41e1639)

$${\color{red}Buyer1 \space enters \space but \space his \space age \space is \space 15!}$$

![buyer2](https://github.com/EnricoAdamelli/DALI-mercato/assets/64257821/b8bdfca3-abc4-486e-82a2-1828a363d918)

$${\color{blue}Market \space output \space (final) :}$$

![mercato](https://github.com/EnricoAdamelli/DALI-mercato/assets/64257821/3f99501b-b509-49ea-b09c-79e9dec96ab4)

# Installation instructions
This project has been implemented in a Windows System, so it is necessary WSL.
To run this project you need the following:
- Sicstus installation (at least v4.6.0) : https://sicstus.sics.se/download4.html
- DALI installation by following this link: https://github.com/AAAI-DISIM-UnivAQ/DALI
- clone this repository on your working directory
- start the program startmas
- in the active user window, write the commands as done in the above example.

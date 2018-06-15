import java.io.File;

import javax.xml.parsers.SAXParser;
import javax.xml.parsers.SAXParserFactory;

import uplatnica.model.Uplatnice;
import uplatnica.parser.UplatnicaSaxParser;

public class MyApp {

	private static SAXParserFactory factory = SAXParserFactory.newInstance();

	public static void main(String[] args) {
		
		String name = "data/Uplatnice.xml";
		try {
			SAXParser saxParser = factory.newSAXParser();
			UplatnicaSaxParser uplatnicaParser = new UplatnicaSaxParser();
			saxParser.parse(new File(name), uplatnicaParser);

			Uplatnice uplatnice = uplatnicaParser.getUplatnice();
			System.out.println(uplatnice);
		} catch (Exception e) {
			e.printStackTrace();
		}
	}

}
public class UplatnicaSaxParser extends DefaultHandler {
	private StringBuffer textBuffer;
	private Uplatnice uplatnice = new Uplatnice();

	private Uplatnica currentUplatnica;
	private Lice currentLice;
	private Adresa currentAdresa;
	private Uplata currentUplata;
	private Racun currentRacun;
	private String text;
	
	public Uplatnice getUplatnice() {
		return uplatnice;
	}

	// poziva se kada pocinje element
	public void startElement(String namespaceURI, // namespace elementa
			String sName, // simple name elementa (ne sadrzi prefix)
			String qName, // qualified name elementa (sadrzi prefix)
			Attributes attrs // atributi elementa
	) throws SAXException {
		if (qName.equalsIgnoreCase("uplatnica")) {
			currentUplatnica = new Uplatnica();
		} else if (qName.equalsIgnoreCase("duznik") || qName.equalsIgnoreCase("primalac")) {
			String lice = attrs.getValue("type");
			if (lice.equalsIgnoreCase("fizicko_lice")) {
				currentLice = new FizickoLice();
			} else if (lice.equalsIgnoreCase("pravno_lice")) {
				currentLice = new PravnoLice();
			}
		} else if (qName.equalsIgnoreCase("adresa")) {
			currentAdresa = new Adresa();
		} else if (qName.equalsIgnoreCase("mesto")) {
			currentAdresa.setPostanskiBroj(Integer.parseInt(attrs.getValue("postanski_broj")));
		} else if (qName.equalsIgnoreCase("uplata")) {
			currentUplata = new Uplata();
			String hitno = attrs.getValue("hitno");
			if (hitno != null && hitno.equalsIgnoreCase("da"))
				currentUplata.setHitno(true);
		} else if (qName.equalsIgnoreCase("racun_duznika") || qName.equalsIgnoreCase("racun_primaoca")) {
			currentRacun = new Racun();
		}
	}

	// za kraj elementa
	public void endElement(String namespaceURI, String sName, String qName)
			throws SAXException {
		
		if (qName.equalsIgnoreCase("uplatnica")) {
			uplatnice.addUplatnica(currentUplatnica);
			currentUplatnica = null;
		} else if (qName.equalsIgnoreCase("duznik")) {
			currentUplatnica.setDuznik(currentLice);
			currentLice = null;
		} else if (qName.equalsIgnoreCase("primalac")) {
			currentUplatnica.setPrimalac(currentLice);
			currentLice = null;
		} else if (qName.equalsIgnoreCase("svrha_placanja")) {
			currentUplatnica.setSvrhaPlacanja(text);
		} else if (qName.equalsIgnoreCase("naziv")) {
			((PravnoLice) currentLice).setNaziv(text);
		} else if (qName.equalsIgnoreCase("ime")) {
			((FizickoLice) currentLice).setIme(text);
		} else if (qName.equalsIgnoreCase("prezime")) {
			((FizickoLice) currentLice).setPrezime(text);
		} else if (qName.equalsIgnoreCase("adresa")) {
			currentLice.setAdresa(currentAdresa);
			currentAdresa = null;
		} else if (qName.equalsIgnoreCase("ulica")) {
			currentAdresa.setUlica(text);
		} else if (qName.equalsIgnoreCase("broj")) {
			currentAdresa.setBroj(Integer.parseInt(text));
		} else if (qName.equalsIgnoreCase("mesto")) {
			currentAdresa.setMesto(text);
		} else if (qName.equalsIgnoreCase("uplata")) {
			currentUplatnica.setUplata(currentUplata);
			currentUplata = null;
		} else if (qName.equalsIgnoreCase("sifra_placanja")) {
			currentUplata.setSifraPlacanja(Integer.parseInt(text));
		} else if (qName.equalsIgnoreCase("sifra_placanja")) {
			currentUplata.setSifraPlacanja(Integer.parseInt(text));
		} else if (qName.equalsIgnoreCase("iznos")) {
			currentUplata.setIznos(Double.parseDouble(text));
		} else if (qName.equalsIgnoreCase("valuta")) {
			currentUplata.setValuta(text);
		} else if (qName.equalsIgnoreCase("racun_duznika")) {
			currentUplata.setRacunDuznika(currentRacun);
			currentRacun = null;
		} else if (qName.equalsIgnoreCase("racun_primaoca")) {
			currentUplata.setRacunPoverioca(currentRacun);
			currentRacun = null;
		} else if (qName.equalsIgnoreCase("racun")) {
			currentRacun.setBrojRacuna(text);
		} else if (qName.equalsIgnoreCase("model")) {
			currentRacun.setModel(Integer.parseInt(text));
		} else if (qName.equalsIgnoreCase("poziv_na_broj")) {
			currentRacun.setPozivNaBroj(text);
		}
	}

	public void characters(char buf[], int offset, int len) throws SAXException {
		text = new String(buf, offset, len);
	}

}



import java.io.File;

import javax.xml.parsers.DocumentBuilder;
import javax.xml.parsers.DocumentBuilderFactory;

import org.w3c.dom.Document;

import uplatnica.Uplatnice;

public class MyApp {

	private static DocumentBuilderFactory factory = DocumentBuilderFactory.newInstance();

	public static void main(String[] args) {
		Document document = null;
		try {
			// Kreira DocumentBuilder
			DocumentBuilder builder = factory.newDocumentBuilder();
			// Parsiramo ulaz, tj. XML file
			document = builder.parse(new File("data/Uplatnice.xml"));
			Uplatnice uplatnice = Uplatnice.loadFromDom(document.getElementsByTagName("uplatnice").item(0));
			System.out.println(uplatnice);
		} catch (Exception e) {
			e.printStackTrace();
		}
	}

}


public static Uplatnica loadFromDom(Node uplatnicaNode){
		Element uplatnicaElement = (Element) uplatnicaNode;
		Node duznikNode = uplatnicaElement.getElementsByTagName("duznik").item(0);
		Node svrhaUplateNode = uplatnicaElement.getElementsByTagName("svrha_placanja").item(0);
		Node primalacNode = uplatnicaElement.getElementsByTagName("primalac").item(0);
		Node uplataNode = uplatnicaElement.getElementsByTagName("uplata").item(0);
		
		Uplatnica rez = new Uplatnica();
		rez.setDuznik(Lice.loadFromDom(duznikNode));
		rez.setSvrhaPlacanja(svrhaUplateNode.getTextContent());
		rez.setPrimalac(Lice.loadFromDom(primalacNode));
		rez.setUplata(Uplata.loadFromDom(uplataNode));
		return rez;
	}

# SDMCOP

classes
public class Product implements Serializable {
    private int productId;
    private String productName;
    private double price;
    private int quantity;
    // constructors, getters/setters
    @Override
    public String toString() { return productId+" "+productName+" "+price+" "+quantity; }
}
public class ElectronicProduct extends Product {
    private boolean warranty;
    @Override
    public String toString() { return super.toString()+" Warranty:"+warranty; }
}
public class GroceryProduct extends Product {
    private String expiryDate;
    @Override
    public String toString() { return super.toString()+" Expiry:"+expiryDate; }
}


dao

public interface ProductDao {
    boolean save(Product p);
    Product findById(int id);
    List<Product> findByName(String name);
    Map<Integer, Product> findAll();
    boolean removeById(int id);
    boolean removeByPrice(double price);
    boolean updatePrice(int id, double price);
    boolean updateQuantity(int id, int qty);
    List<Product> sortByName();
    List<Product> sortByPrice();
}

daoimpl
public class ProductDaoImpl implements ProductDao {
    private Map<Integer, Product> pmap;
    private final String FILE_NAME="products.dat";

    public ProductDaoImpl() {
        pmap = loadFromFile();
        if(pmap==null) pmap=new HashMap<>();
    }

    @Override
    public boolean save(Product p) { pmap.put(p.getProductId(), p); saveToFile(); return true; }

    @Override
    public Product findById(int id) { return pmap.get(id); }

    @Override
    public List<Product> findByName(String name) {
        List<Product> list = new ArrayList<>();
        for(Product p: pmap.values()) if(p.getProductName().equalsIgnoreCase(name)) list.add(p);
        return list.size()>0? list:null;
    }

    @Override
    public Map<Integer, Product> findAll() { return pmap; }

    @Override
    public boolean removeById(int id) { if(pmap.remove(id)!=null){ saveToFile(); return true;} return false; }

    @Override
    public boolean removeByPrice(double price) {
        boolean removed=false;
        Iterator<Product> it = pmap.values().iterator();
        while(it.hasNext()){ Product p=it.next(); if(p.getPrice()>price){ it.remove(); removed=true; }}
        if(removed) saveToFile(); return removed;
    }

    @Override
    public boolean updatePrice(int id, double price){ Product p=pmap.get(id); if(p!=null){ p.setPrice(price); saveToFile(); return true;} return false; }

    @Override
    public boolean updateQuantity(int id, int qty){ Product p=pmap.get(id); if(p!=null){ p.setQuantity(qty); saveToFile(); return true;} return false; }

    @Override
    public List<Product> sortByName(){ List<Product> list=new ArrayList<>(pmap.values()); list.sort(Comparator.comparing(Product::getProductName)); return list; }

    @Override
    public List<Product> sortByPrice(){ List<Product> list=new ArrayList<>(pmap.values()); list.sort(Comparator.comparingDouble(Product::getPrice)); return list; }

    // File handling
    private void saveToFile(){
        try(ObjectOutputStream oos=new ObjectOutputStream(new FileOutputStream(FILE_NAME))){ oos.writeObject(pmap); } catch(Exception e){ e.printStackTrace();}
    }
    @SuppressWarnings("unchecked")
    private Map<Integer, Product> loadFromFile(){
        try(ObjectInputStream ois=new ObjectInputStream(new FileInputStream(FILE_NAME))){ return (Map<Integer, Product>)ois.readObject(); } catch(Exception e){ return null; }
    }
}


service

public interface ProductService {
    boolean addNewProduct();
    Map<Integer, Product> displayAll();
    Product displayById(int id);
    List<Product> displayByName(String name);
    boolean deleteById(int id);
    boolean deleteByPrice(double price);
    boolean updatePrice(int id, double price);
    boolean updateQuantity(int id, int qty);
    List<Product> sortByName();
    List<Product> sortByPrice();
}

serviceimpl
public class ProductServiceImpl implements ProductService {
    private ProductDao pdao=new ProductDaoImpl();
    private Scanner sc=new Scanner(System.in);

    public boolean addNewProduct(){
        System.out.print("Type(1-Elec/2-Grocery):"); int type=sc.nextInt();
        System.out.print("Id:"); int id=sc.nextInt();
        System.out.print("Name:"); String name=sc.next();
        System.out.print("Price:"); double price=sc.nextDouble();
        System.out.print("Qty:"); int qty=sc.nextInt();

        Product p=null;
        if(type==1){ System.out.print("Warranty(true/false):"); boolean w=sc.nextBoolean(); p=new ElectronicProduct(id,name,price,qty,w);}
        else if(type==2){ System.out.print("Expiry(dd/mm/yyyy):"); String e=sc.next(); p=new GroceryProduct(id,name,price,qty,e);}
        else{ System.out.println("Invalid type"); return false;}
        return pdao.save(p);
    }

    public Map<Integer, Product> displayAll(){ return pdao.findAll(); }
    public Product displayById(int id){ return pdao.findById(id); }
    public List<Product> displayByName(String name){ return pdao.findByName(name); }
    public boolean deleteById(int id){ return pdao.removeById(id); }
    public boolean deleteByPrice(double price){ return pdao.removeByPrice(price); }
    public boolean updatePrice(int id,double price){ return pdao.updatePrice(id,price); }
    public boolean updateQuantity(int id,int qty){ return pdao.updateQuantity(id,qty); }
    public List<Product> sortByName(){ return pdao.sortByName(); }
    public List<Product> sortByPrice(){ return pdao.sortByPrice(); }
}



test
public class ProductTest{
    public static void main(String[] args){
        ProductService service=new ProductServiceImpl();
        Scanner sc=new Scanner(System.in);
        int ch;
        do{
            System.out.println("\n1.Add 2.All 3.ById 4.ByName 5.DelId 6.DelPrice 7.UpPrice 8.UpQty 9.SortName 10.SortPrice 0.Exit");
            ch=sc.nextInt();
            switch(ch){
                case 1: service.addNewProduct(); break;
                case 2: service.displayAll().values().forEach(System.out::println); break;
                case 3: System.out.println(service.displayById(sc.nextInt())); break;
                case 4: service.displayByName(sc.next()).forEach(System.out::println); break;
                case 5: service.deleteById(sc.nextInt()); break;
                case 6: service.deleteByPrice(sc.nextDouble()); break;
                case 7: service.updatePrice(sc.nextInt(), sc.nextDouble()); break;
                case 8: service.updateQuantity(sc.nextInt(), sc.nextInt()); break;
                case 9: service.sortByName().forEach(System.out::println); break;
                case 10: service.sortByPrice().forEach(System.out::println); break;
                case 0: System.out.println("Thanks"); break;
            }
        }while(ch!=0);
        sc.close();
    }
}


import java.time.LocalDateTime;
import java.util.*;

// ================= PASSENGER CLASS =================
class Passenger {
    private String paxId;
    private String name;
    private String flightNo;
    private String contact;
    private List<Claim> claims;

    public Passenger(String paxId, String name, String flightNo, String contact) {
        this.paxId = paxId;
        this.name = name;
        this.flightNo = flightNo;
        this.contact = contact;
        this.claims = new ArrayList<>();
    }

    public String getPaxId() { return paxId; }
    public String getName() { return name; }
    public String getFlightNo() { return flightNo; }
    public String getContact() { return contact; }
    public List<Claim> getClaims() { return claims; }

    public void addClaim(Claim c) {
        claims.add(c);
    }

    @Override
    public String toString() {
        return "Passenger{" +
                "paxId='" + paxId + '\'' +
                ", name='" + name + '\'' +
                ", flightNo='" + flightNo + '\'' +
                ", contact='" + contact + '\'' +
                '}';
    }
}

// ================= BAGGAGE CLASS =================
class Baggage {
    private String bagTag;
    private double weight;
    private Passenger owner;
    private List<String> routeHistory;
    private String status;

    public Baggage(String bagTag, double weight, Passenger owner) {
        this.bagTag = bagTag;
        this.weight = weight;
        this.owner = owner;
        this.status = "Registered";
        this.routeHistory = new ArrayList<>();
    }

    public String getBagTag() { return bagTag; }
    public double getWeight() { return weight; }
    public Passenger getOwner() { return owner; }
    public String getStatus() { return status; }

    public void setStatus(String status) {
        this.status = status;
    }

    public void addRouteLog(String log) {
        routeHistory.add(log);
    }

    public List<String> getRouteHistory() {
        return routeHistory;
    }

    @Override
    public String toString() {
        return "Baggage{" +
                "bagTag='" + bagTag + '\'' +
                ", weight=" + weight +
                ", owner=" + owner.getName() +
                ", status='" + status + '\'' +
                '}';
    }
}

// ================= CHECKPOINT CLASS =================
class Checkpoint {
    private String id;
    private String name;
    private LocalDateTime timestamp;

    public Checkpoint(String id, String name) {
        this.id = id;
        this.name = name;
        this.timestamp = LocalDateTime.now();
    }

    public String getId() { return id; }
    public String getName() { return name; }
    public LocalDateTime getTimestamp() { return timestamp; }

    @Override
    public String toString() {
        return "Checkpoint{" +
                "id='" + id + '\'' +
                ", name='" + name + '\'' +
                ", timestamp=" + timestamp +
                '}';
    }
}

// ================= CLAIM BASE CLASS =================
abstract class Claim {
    protected String claimId;
    protected Baggage bag;
    protected String description;
    protected double payout;

    public Claim(String claimId, Baggage bag, String description) {
        this.claimId = claimId;
        this.bag = bag;
        this.description = description;
    }

    public abstract void settleClaim();

    public String getClaimId() { return claimId; }
    public double getPayout() { return payout; }
    public String getDescription() { return description; }

    @Override
    public String toString() {
        return "Claim{" +
                "claimId='" + claimId + '\'' +
                ", bagTag='" + bag.getBagTag() + '\'' +
                ", description='" + description + '\'' +
                ", payout=" + payout +
                '}';
    }
}

// ================= LOSS CLAIM (Inheritance + Override) =================
class LossClaim extends Claim {
    public LossClaim(String claimId, Baggage bag, String description) {
        super(claimId, bag, description);
    }

    @Override
    public void settleClaim() {
        payout = bag.getWeight() * 100; // Example payout calculation
    }
}

// ================= DAMAGE CLAIM (Inheritance + Override) =================
class DamageClaim extends Claim {
    public DamageClaim(String claimId, Baggage bag, String description) {
        super(claimId, bag, description);
    }

    @Override
    public void settleClaim() {
        payout = bag.getWeight() * 50; // Example payout calculation
    }
}

// ================= BAGGAGE SERVICE CLASS =================
class BaggageService {
    private Map<String, Baggage> bagRegistry;

    public BaggageService() {
        bagRegistry = new HashMap<>();
    }

    public void registerBag(Baggage bag) {
        bagRegistry.put(bag.getBagTag(), bag);
        bag.addRouteLog("Bag registered at: " + LocalDateTime.now());
        System.out.println("Bag registered: " + bag);
    }

    // Method Overloading: updateMovement by ID
    public void updateMovement(String bagTag, String checkpointId) {
        Baggage bag = bagRegistry.get(bagTag);
        if (bag != null) {
            String log = "Moved via checkpoint " + checkpointId + " at " + LocalDateTime.now();
            bag.addRouteLog(log);
            bag.setStatus("In Transit");
        }
    }

    // Method Overloading: updateMovement by object + time
    public void updateMovement(String bagTag, Checkpoint cp, LocalDateTime time) {
        Baggage bag = bagRegistry.get(bagTag);
        if (bag != null) {
            String log = "Checkpoint: " + cp.getName() + " at " + time;
            bag.addRouteLog(log);
            bag.setStatus("At " + cp.getName());
        }
    }

    public void locateBag(String bagTag) {
        Baggage bag = bagRegistry.get(bagTag);
        if (bag != null) {
            System.out.println("Location history for bag " + bagTag + ": " + bag.getRouteHistory());
        }
    }

    public void raiseClaim(Passenger pax, Claim claim) {
        claim.settleClaim();  // Polymorphism: different settlement logic
        pax.addClaim(claim);
        System.out.println("Claim processed: " + claim);
    }
}

// ================= MAIN APP =================
public class BaggageAppMain {
    public static void main(String[] args) {
        // Create passenger
        Passenger p1 = new Passenger("P001", "Khaveenaa", "AI101", "9999999999");

        // Create baggage
        Baggage b1 = new Baggage("B001", 20.5, p1);

        // Service
        BaggageService service = new BaggageService();
        service.registerBag(b1);

        // Update movements
        Checkpoint c1 = new Checkpoint("C1", "Check-in");
        Checkpoint c2 = new Checkpoint("C2", "Security");
        Checkpoint c3 = new Checkpoint("C3", "Loading");

        service.updateMovement("B001", "C1");
        service.updateMovement("B001", c2, LocalDateTime.now());
        service.updateMovement("B001", c3, LocalDateTime.now());

        // Locate bag
        service.locateBag("B001");

        // Raise claim (loss)
        Claim lossClaim = new LossClaim("CL001", b1, "Bag lost in transit");
        service.raiseClaim(p1, lossClaim);

        // Raise claim (damage)
        Claim damageClaim = new DamageClaim("CL002", b1, "Handle broken");
        service.raiseClaim(p1, damageClaim);

        // Print passenger claims
        System.out.println("Claims raised by passenger: " + p1.getClaims());
    }
}

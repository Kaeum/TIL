# 01 객체, 설계



   * 절차지향적 코드

     ```java
     public class Invitation{
         private LocalDateTime when;
     }
     ```

     ```java
     public class Ticket{
     	private Long fee;
         
         public Long getFee(){
             return fee;
         }
     }
     ```

     ```java
     public class Bag{
         private Long amount;
         private Invitation invitation;
         private Ticket ticket;
         
         public Bag(long amount){
             this(null, amount);
         }
         
         public Bag(Invitation invitation, long amount){
             this.invitation = invitation;
             this.amount = amount;
         }
         
         public boolean hasInvitation(){
             return invitation != null;
         }
         
         public boolean hasTicket(){
             return ticket != null;
         }
         
         public void setTicket(Ticket ticket){
             this.ticket = ticket;
         }
         
         puiblic void minusAmount(Long amount){
             this.amount -= amount;
         }
         
         public void plusAmount(Long amount){
             this.amount += amount;
         }
     }
     ```

     ```java
     public class Audience{
         private Bag bag;
         
         public Audience(Bag bag){
             this.bag = bag;
         }
         
         public Bag getBag(){
             return bag;
         }
     }
     ```

     ```java
     public class TicketOffice{
         private Long amount;
         private List<Ticket> tickets = new ArrayList<>();
         
         public TicketOffice(Long amount, Ticket ... tickets){
             this.amount = amount;
             this.tickets.addAll(Arrays.asList(tickets));
         }
         
         public Ticket getTicket(){
             return tickets.remove(0);
         }
         
         public void minusAmount(Long amount){
             this.amount -= amount;
         }
         
         public void plusAmount(Long amount){
             this.amount += amount;
         }
     }
     ```

     ```java
     public class TicketSeller{
         private TicketOffice ticketOffice;
         
         public TicketSeller(TicketOffice ticketOffice){
             this.ticketOffice = ticketOffice;
         }
         
         public TicketOffice getTicketOffice(){
             return ticketOffice
         }
     }
     ```

     ```java
     public class Theater{
         private TicketSeller ticketSeller;
         
         public Theater(TicketSeller ticketSeller){
             this.ticketSeller = ticketSeller;
         }
         
         public void enter(Audience audience){
             if(audience.getBag().hasInvitation()){
                 Ticket ticket = ticketSeller.getTicketOffice().getTicket();
                 audience.getBag().setTicket(ticket);
             } else{
                 Ticket ticket = ticketSeller.getTicketOffice().getTicket();
                 audience.getBag().minusAmount(ticket.getFee());
                 ticketSeller.getTicketOffice().plusAmount(ticket.getFee());
                 audience.getBag().setTicket(ticket);
             }
         }
     }
     ```

     * dependency : 
	       * Audience -> Bag
	       * Bag -> Invitation 
	       * Bag -> Ticket
	       * TicketSeller -> TicketOffice
	       * TicketOffice -> Ticket
	       * Theater -> TicketSeller
	       * Theater -> TicketOffice
	       * Theater -> Audience
	       * Theater -> Ticket
     * 위 설계의 문제점
	       1. 변경 용이성이 떨어진다.
	       2. 독자입장에서의 가독성이 떨어진다.

   * 설계 개선하기

     ```java
     public class Invitation{
         private LocalDateTime when;
     }
     ```

     ```java
     public class Ticket{
     	private Long fee;
         
         public Long getFee(){
             return fee;
         }
     }
     ```

     ```java
     public class Bag{
         private Long amount;
         private Invitation invitation;
         private Ticket ticket;
         
         public Bag(long amount){
             this(null, amount);
         }
         
         public Bag(Invitation invitation, long amount){
             this.invitation = invitation;
             this.amount = amount;
         }
         
         public boolean hasInvitation(){
             return invitation != null;
         }
         
         public boolean hasTicket(){
             return ticket != null;
         }
         
         public void setTicket(Ticket ticket){
             this.ticket = ticket;
         }
         
         puiblic void minusAmount(Long amount){
             this.amount -= amount;
         }
         
         public void plusAmount(Long amount){
             this.amount += amount;
         }
     }
     ```

     ```java
     public class Audience{
         private Bag bag;
         
         public Audience(Bag bag){
             this.bag = bag;
         }
            
         public Long buy(Ticket ticket){
             
         	if(!bag.hasInvitation())
                 bag.minusAmount(ticket.getFee())        
     
             return bag.hasInvitation() ? 0L : ticket.getFee();
         }
     }
     ```

     ```java
     public class TicketOffice{
         private Long amount;
         private List<Ticket> tickets = new ArrayList<>();
         
         public TicketOffice(Long amount, Ticket ... tickets){
             this.amount = amount;
             this.tickets.addAll(Arrays.asList(tickets));
         }
         
         public Ticket getTicket(){
             return tickets.remove(0);
         }
         
         public void minusAmount(Long amount){
             this.amount -= amount;
         }
         
         public void plusAmount(Long amount){
             this.amount += amount;
         }
     }
     ```

     ```java
     public class TicketSeller{
         private TicketOffice ticketOffice;
         
         public TicketSeller(TicketOffice ticketOffice){
             this.ticketOffice = ticketOffice;
         }
                
         public void sellTo(Audience audience){
             Ticket ticket = ticketOffice.getTicket();
             ticketOffice.plusamount(audience.buy(ticket));        
         }
         
     }
     ```

     ```java
     public class Theater{
         private TicketSeller ticketSeller;
         
         public Theater(TicketSeller ticketSeller){
             this.ticketSeller = ticketSeller;
         }
         
         public void enter(Audience audience){
     		ticketSeller.sellTo(audience)
         }
     }
     ```

     * dependency : 
	       * Audience -> Bag
	       * Bag -> Invitation 
	       * Bag -> Ticket
	       * TicketSeller -> Audience
	       * TicketSeller -> TicketOffice
	       * TicketSeller -> Ticket
	       * TicketOffice -> Ticket
	       * Theater -> TicketSeller
	       * Theater -> Audience

   
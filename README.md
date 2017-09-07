private static final AccountManagementService accountService = new AccountManagementServiceImpl();

    @Path("account/create")
    @POST
    public Response createAccount(@QueryParam("balance") String balance) {
        Account account = null;
        if(balance == null){
            account = accountService.create();
        }else{
            try{
                BigDecimal amount = new BigDecimal(balance);
                account = accountService.create(amount);

            }catch (NumberFormatException e){
                Response.status(Response.Status.BAD_REQUEST).entity("wrong number format, "+balance).build();
            }
        }
        return Response
            .status(Response.Status.CREATED)
            .entity("Account number created "+account.getAccountNumber()+" with balance "+account.getBalance())
            .build();
    }

    @Path("account/find/{accountNumber}")
    @GET
    public Response findAccount(@PathParam("accountNumber") String accountNumber){
        try{
            Optional<Account> account = accountService.find(Long.parseLong(accountNumber));
            if(account.isPresent()){
                return Response.ok("Account "+account.get().getAccountNumber()+",Balance "+account.get().getBalance()).build();
            }else{
                return Response.status(Response.Status.NOT_FOUND).entity("Account does not exist").build();
            }

        }catch(NumberFormatException e){
            return Response.status(Response.Status.BAD_REQUEST).entity("wrong number format, "+accountNumber).build();
        }catch(Exception e){
            return Response.status(Response.Status.BAD_REQUEST).entity(e.getMessage()).build();
        }
    }

    @Path("account/transfer")
    @PUT
    public Response transfer(@QueryParam("from") String from,@QueryParam("to") String to,@QueryParam("amount") String amount){
        try{
            final long fromAccountId = Long.parseLong(from);
            final long toAccountId = Long.parseLong(to);
            final BigDecimal value = new BigDecimal(amount);
            accountService.transfer(fromAccountId,toAccountId,value);
            return Response.ok("Transfer from "+from+" to "+to+" for "+amount+" successful").build();
        }catch(AccountNotFoundException e){
            return Response.status(Response.Status.NOT_FOUND).entity(e.getMessage()).build();
        }catch(Exception e){
            return Response.status(Response.Status.BAD_REQUEST).entity(e.getMessage()).build();
        }
    }
    
    private final Map<Long, Account> accounts = new ConcurrentHashMap<Long, Account>();
    private final ReadWriteLock lock = new ReentrantReadWriteLock();
    private final AtomicLong accountNumberGenerator = new AtomicLong(10);

    public Account create(BigDecimal withBalance)
    {
        long accountNumber = accountNumberGenerator.incrementAndGet();
        Account account = new Account(accountNumber,withBalance);
        accounts.put(accountNumber,account);
        return account;
    }

    public void delete(long accountNumber)
    {
        try
        {
            lock.writeLock().lock();
            if(!accounts.containsKey(accountNumber)){
                throw new AccountNotFoundException(accountNumber);
            }
            accounts.remove(accountNumber);
        }
        finally
        {
            lock.writeLock().unlock();
        }
    }

    public Account find(long accountNumber)
    {
        try{
            lock.readLock().lock();
            return accounts.getOrDefault(accountNumber,null);
        }finally{
            lock.readLock().unlock();
        }
    }

    public void transfer(long fromAccount, long toAccount, BigDecimal amount)
    {
        try{
            lock.writeLock().lock();
            if(!accounts.containsKey(fromAccount)){
                throw new AccountNotFoundException(fromAccount);
            }
            if(!accounts.containsKey(toAccount)){
                throw new AccountNotFoundException(toAccount);
            }
            Account sourceAccount = accounts.get(fromAccount);
            if(sourceAccount.isBalanceBelow(amount)){
                throw new NotEnoughBalanceException(fromAccount);
            }
            Account destinationAccount = accounts.get(toAccount);
            sourceAccount.debit(amount);
            destinationAccount.credit(amount);
        }finally
        {
            lock.writeLock().unlock();
        }
    }private final Map<Long, Account> accounts = new ConcurrentHashMap<Long, Account>();
    private final ReadWriteLock lock = new ReentrantReadWriteLock();
    private final AtomicLong accountNumberGenerator = new AtomicLong(10);

    public Account create(BigDecimal withBalance)
    {
        long accountNumber = accountNumberGenerator.incrementAndGet();
        Account account = new Account(accountNumber,withBalance);
        accounts.put(accountNumber,account);
        return account;
    }

    public void delete(long accountNumber)
    {
        try
        {
            lock.writeLock().lock();
            if(!accounts.containsKey(accountNumber)){
                throw new AccountNotFoundException(accountNumber);
            }
            accounts.remove(accountNumber);
        }
        finally
        {
            lock.writeLock().unlock();
        }
    }

    public Account find(long accountNumber)
    {
        try{
            lock.readLock().lock();
            return accounts.getOrDefault(accountNumber,null);
        }finally{
            lock.readLock().unlock();
        }
    }

    public void transfer(long fromAccount, long toAccount, BigDecimal amount)
    {
        try{
            lock.writeLock().lock();
            if(!accounts.containsKey(fromAccount)){
                throw new AccountNotFoundException(fromAccount);
            }
            if(!accounts.containsKey(toAccount)){
                throw new AccountNotFoundException(toAccount);
            }
            Account sourceAccount = accounts.get(fromAccount);
            if(sourceAccount.isBalanceBelow(amount)){
                throw new NotEnoughBalanceException(fromAccount);
            }
            Account destinationAccount = accounts.get(toAccount);
            sourceAccount.debit(amount);
            destinationAccount.credit(amount);
        }finally
        {
            lock.writeLock().unlock();
        }
    }private final Map<Long, Account> accounts = new ConcurrentHashMap<Long, Account>();
    private final ReadWriteLock lock = new ReentrantReadWriteLock();
    private final AtomicLong accountNumberGenerator = new AtomicLong(10);

    public Account create(BigDecimal withBalance)
    {
        long accountNumber = accountNumberGenerator.incrementAndGet();
        Account account = new Account(accountNumber,withBalance);
        accounts.put(accountNumber,account);
        return account;
    }

    public void delete(long accountNumber)
    {
        try
        {
            lock.writeLock().lock();
            if(!accounts.containsKey(accountNumber)){
                throw new AccountNotFoundException(accountNumber);
            }
            accounts.remove(accountNumber);
        }
        finally
        {
            lock.writeLock().unlock();
        }
    }

    public Account find(long accountNumber)
    {
        try{
            lock.readLock().lock();
            return accounts.getOrDefault(accountNumber,null);
        }finally{
            lock.readLock().unlock();
        }
    }

    public void transfer(long fromAccount, long toAccount, BigDecimal amount)
    {
        try{
            lock.writeLock().lock();
            if(!accounts.containsKey(fromAccount)){
                throw new AccountNotFoundException(fromAccount);
            }
            if(!accounts.containsKey(toAccount)){
                throw new AccountNotFoundException(toAccount);
            }
            Account sourceAccount = accounts.get(fromAccount);
            if(sourceAccount.isBalanceBelow(amount)){
                throw new NotEnoughBalanceException(fromAccount);
            }
            Account destinationAccount = accounts.get(toAccount);
            sourceAccount.debit(amount);
            destinationAccount.credit(amount);
        }finally
        {
            lock.writeLock().unlock();
        }
    }
    
    private final AccountManagementDao accountDao = new InMemoryAccountManagementDaoImpl();

    public Account create()
    {
        return create(ZERO);
    }

    public Account create(BigDecimal withBalance)
    {
        return accountDao.create(withBalance);
    }

    public Optional<Account> find(long accountNumber)
    {
        return ofNullable(accountDao.find(accountNumber));
    }

    public void delete(long accountNumber)
    {
        accountDao.delete(accountNumber);
    }

    public void transfer(long fromAccount, long toAccount, BigDecimal amount)
    {
        if(ZERO.compareTo(amount) >= 0){
            throw new IllegalArgumentException("Transfer amount must be more than zero");
        }
        if(fromAccount == toAccount){
            throw new IllegalArgumentException("Source and Destination accounts can't be same");
        }
        accountDao.transfer(fromAccount,toAccount,amount);
    }
    
    
    

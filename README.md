#FluentCommand

Fluent Wrapper for DbCommand.

##Download

The FluentCommand library is available on nuget.org via package name `FluentCommand`.

To install FluentCommand, run the following command in the Package Manager Console

    PM> Install-Package FluentCommand
    
More information about NuGet package avaliable at
<https://nuget.org/packages/FluentCommand>

##Development Builds


Development builds are available on the myget.org feed.  A development build is promoted to the main NuGet feed when it's determined to be stable. 

In your Package Manager settings add the following package source for development builds:
<http://www.myget.org/F/loresoft/>

##Features

- Fluent wrapper over DbConnection and DbCommand
- Callback for parameter return values
- Automatic handling of connection state
- Caching of results
- Automatic creating of entity from DataReader
- Create Dynamic objects from DataReader
- Handles multiple result sets


##Example

Query all users with email domain.  Entity is automaticly created from DataReader.

    string email = "%@battlestar.com";
    string sql = "select * from [User] where EmailAddress like @EmailAddress";

    List<User> users;
    using (var session = new DataSession("Tracker").Log(Console.WriteLine))
    {
        users = session            
            .Sql(sql)
            .Parameter("@EmailAddress", email)
            .Query<User>();
    }

Execute a stored procedure with out parameters

    Guid userId = Guid.Empty;
    int errorCode = -1;

    var username = "test." + DateTime.Now.Ticks;
    var email = username + "@email.com";

    int result;
    using (var session = new DataSession("AspNet").Log(Console.WriteLine))
    {
        result = session.StoredProcedure("[dbo].[aspnet_Membership_CreateUser]")
            .Parameter("@ApplicationName", "/")
            .Parameter("@UserName", username)
            .Parameter("@Password", "T@est" + DateTime.Now.Ticks)
            .Parameter("@Email", email)
            .Parameter("@PasswordSalt", "test salt")
            .Parameter<string>("@PasswordQuestion", null)
            .Parameter<string>("@PasswordAnswer", null)
            .Parameter("@IsApproved", true)
            .Parameter("@CurrentTimeUtc", DateTime.UtcNow)
            .Parameter("@UniqueEmail", 1)
            .Parameter("@PasswordFormat", 1)
            .ParameterOut<Guid>("@UserId", p => userId = p)
            .Return<int>(p => errorCode = p)
            .Execute();
    }

Query for user by email address.  Also return Role and Status entities.

    string email = "kara.thrace@battlestar.com";
    string sql = "select * from [User] where EmailAddress = @EmailAddress; " +
                 "select * from [Status]; " +
                 "select * from [Priority]; ";

    User user = null;
    List<Status> status = null;
    List<Priority> priorities = null;

    using (var session = new DataSession("Tracker").Log(Console.WriteLine))
    {
        session.Sql(sql)
            .Parameter("@EmailAddress", email)
            .QueryMultiple(q =>
            {
                user = q.QuerySingle<User>();
                status = q.Query<Status>().ToList();
                priorities = q.Query<Priority>().ToList();
            });
    }

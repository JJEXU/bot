import discord
from discord import app_commands
import asyncio
import customtkinter as ctk
import random

TOKEN = ""  # Replace this with your new bot token
LOG_CHANNEL_ID = 1358031747958313062  # Replace with your actual staff applications channel ID


class StaffApplicationBot(discord.Client):

    def __init__(self):
        intents = discord.Intents.default()
        intents.members = True
        intents.message_content = True
        super().__init__(intents=intents)
        self.tree = app_commands.CommandTree(self)

    async def setup_hook(self):
        print(f"✅ Logged in as {self.user}")
        await self.tree.sync()  # Sync the slash commands
        print("✅ Slash commands synced!")

        # Start GUI in a separate thread so it doesn't block bot execution
        asyncio.create_task(self.start_gui())

    async def start_gui(self):
        gui = ApplicationGUI(self)
        gui.start()


bot = StaffApplicationBot()


# Slash command to apply for staff
@bot.tree.command(name="serverinfo",
                  description="Get information about the server")
async def serverinfo(interaction: discord.Interaction):
    guild = interaction.guild
    embed = discord.Embed(title=f"{guild.name} Server Information",
                          color=discord.Color.blue())

    # Safely get owner info
    owner_value = "Ben"
    if guild.owner:
        owner_value = guild.owner.mention

    embed.add_field(name="Owner", value=owner_value, inline=True)
    embed.add_field(name="Members", value=guild.member_count, inline=True)
    embed.add_field(name="Created At",
                    value=guild.created_at.strftime("%Y-%m-%d"),
                    inline=True)

    if guild.icon:
        embed.set_thumbnail(url=guild.icon.url)

    await interaction.response.send_message(embed=embed)


@bot.tree.command(name="userinfo", description="Get information about a user")
async def userinfo(interaction: discord.Interaction,
                   member: discord.Member = None):
    member = member or interaction.user
    embed = discord.Embed(title=f"User Information - {member.name}",
                          color=discord.Color.green())
    embed.add_field(name="Joined Server",
                    value=member.joined_at.strftime("%Y-%m-%d"),
                    inline=True)
    embed.add_field(name="Account Created",
                    value=member.created_at.strftime("%Y-%m-%d"),
                    inline=True)
    embed.add_field(name="Top Role",
                    value=member.top_role.mention,
                    inline=True)
    embed.set_thumbnail(url=member.avatar.url if member.avatar else None)
    await interaction.response.send_message(embed=embed)


@bot.tree.command(name="8ball", description="Ask the magic 8ball a question")
async def eightball(interaction: discord.Interaction, question: str):
    import random
    responses = [
        "It is certain.", "Without a doubt.", "Yes definitely.",
        "You may rely on it.", "Ask again later.", "Cannot predict now.",
        "Better not tell you now.", "Don't count on it.", "My reply is no.",
        "My sources say no.", "Very doubtful."
    ]
    response = random.choice(responses)
    embed = discord.Embed(title="🎱 Magic 8Ball", color=discord.Color.purple())
    embed.add_field(name="Question", value=question, inline=False)
    embed.add_field(name="Answer", value=response, inline=False)
    await interaction.response.send_message(embed=embed)


@bot.tree.command(name="flip", description="Flip a coin")
async def flip(interaction: discord.Interaction):
    import random
    result = random.choice(["Heads", "Tails"])
    embed = discord.Embed(title="🪙 Coin Flip",
                          description=f"Result: **{result}**",
                          color=discord.Color.gold())
    await interaction.response.send_message(embed=embed)


@bot.tree.command(name="roll", description="Roll a dice")
async def roll(interaction: discord.Interaction, sides: int = 6):
    import random
    if sides < 2:
        await interaction.response.send_message(
            "The dice must have at least 2 sides!", ephemeral=True)
        return
    result = random.randint(1, sides)
    embed = discord.Embed(
        title="🎲 Dice Roll",
        description=f"Rolling a {sides}-sided dice...\nResult: **{result}**",
        color=discord.Color.blue())
    await interaction.response.send_message(embed=embed)


@bot.tree.command(name="quote", description="Get a random inspirational quote")
async def quote(interaction: discord.Interaction):
    quotes = [
        "Be yourself; everyone else is already taken. - Oscar Wilde",
        "The only way to do great work is to love what you do. - Steve Jobs",
        "Life is what happens when you're busy making other plans. - John Lennon",
        "Success is not final, failure is not fatal. - Winston Churchill",
        "The best way to predict the future is to create it. - Peter Drucker"
    ]
    quote = random.choice(quotes)
    embed = discord.Embed(title="📜 Random Quote",
                          description=quote,
                          color=discord.Color.gold())
    await interaction.response.send_message(embed=embed)


@bot.tree.command(name="joke", description="Get a random joke")
async def joke(interaction: discord.Interaction):
    jokes = [
        "Why don't scientists trust atoms? Because they make up everything!",
        "What do you call a bear with no teeth? A gummy bear!",
        "Why did the scarecrow win an award? He was outstanding in his field!",
        "What do you call a fake noodle? An impasta!",
        "Why did the cookie go to the doctor? Because it was feeling crumbly!"
    ]
    joke = random.choice(jokes)
    embed = discord.Embed(title="😄 Random Joke",
                          description=joke,
                          color=discord.Color.green())
    await interaction.response.send_message(embed=embed)


@bot.tree.command(name="rps", description="Play rock, paper, scissors")
async def rps(interaction: discord.Interaction, choice: str):
    choices = ["rock", "paper", "scissors"]
    if choice.lower() not in choices:
        await interaction.response.send_message(
            "Please choose rock, paper, or scissors!", ephemeral=True)
        return

    bot_choice = random.choice(choices)
    user_choice = choice.lower()

    if user_choice == bot_choice:
        result = "It's a tie!"
    elif (user_choice == "rock" and bot_choice == "scissors") or \
         (user_choice == "paper" and bot_choice == "rock") or \
         (user_choice == "scissors" and bot_choice == "paper"):
        result = "You win!"
    else:
        result = "I win!"

    embed = discord.Embed(title="🎮 Rock, Paper, Scissors",
                          color=discord.Color.blue())
    embed.add_field(name="Your Choice",
                    value=user_choice.capitalize(),
                    inline=True)
    embed.add_field(name="My Choice",
                    value=bot_choice.capitalize(),
                    inline=True)
    embed.add_field(name="Result", value=result, inline=False)
    await interaction.response.send_message(embed=embed)


class PollModal(discord.ui.Modal, title="Create a Poll"):
    question = discord.ui.TextInput(label="Poll Question",
                                    placeholder="What would you like to ask?",
                                    style=discord.TextStyle.paragraph,
                                    required=True,
                                    max_length=1000)

    option1 = discord.ui.TextInput(label="Option 1",
                                   placeholder="Enter first option",
                                   required=True,
                                   max_length=100)

    option2 = discord.ui.TextInput(label="Option 2",
                                   placeholder="Enter second option",
                                   required=True,
                                   max_length=100)

    async def on_submit(self, interaction: discord.Interaction):
        embed = discord.Embed(title="📊 Poll",
                              description=self.question.value,
                              color=discord.Color.purple())
        embed.add_field(name="Option 1️⃣",
                        value=self.option1.value,
                        inline=False)
        embed.add_field(name="Option 2️⃣",
                        value=self.option2.value,
                        inline=False)
        embed.set_footer(text="React with the corresponding number to vote!")

        await interaction.response.send_message(embed=embed)
        msg = await interaction.original_response()
        await msg.add_reaction("1️⃣")
        await msg.add_reaction("2️⃣")


@bot.tree.command(name="poll", description="Create a poll with custom options")
async def poll(interaction: discord.Interaction):
    await interaction.response.send_modal(PollModal())


@bot.tree.command(name="fact", description="Get a random fun fact")
async def fact(interaction: discord.Interaction):
    facts = [
        "Honey never spoils. Archaeologists have found pots of honey in ancient Egyptian tombs that are over 3,000 years old!",
        "A day on Venus is longer than its year. It takes Venus 243 Earth days to rotate on its axis!",
        "The first oranges weren't orange! The original oranges from Southeast Asia were actually green.",
        "A cat has 32 muscles in each ear.",
        "A group of flamingos is called a 'flamboyance'."
    ]
    fact = random.choice(facts)
    embed = discord.Embed(title="🎓 Random Fact",
                          description=fact,
                          color=discord.Color.blue())
    await interaction.response.send_message(embed=embed)


@bot.tree.command(name="avatar", description="Get a user's avatar URL")
async def avatar(interaction: discord.Interaction,
                 member: discord.Member = None):
    member = member or interaction.user
    embed = discord.Embed(title=f"{member.name}'s Avatar",
                          color=discord.Color.blue())
    embed.set_image(
        url=member.avatar.url if member.avatar else member.default_avatar.url)
    await interaction.response.send_message(embed=embed)


@bot.tree.command(name="ping", description="Check bot's latency")
async def ping(interaction: discord.Interaction):
    latency = round(bot.latency * 1000)
    embed = discord.Embed(title="🏓 Pong!",
                          description=f"Latency: {latency}ms",
                          color=discord.Color.green())
    await interaction.response.send_message(embed=embed)


@bot.tree.command(name="remind", description="Set a reminder (in minutes)")
async def remind(interaction: discord.Interaction, minutes: int,
                 reminder: str):
    if minutes <= 0:
        await interaction.response.send_message(
            "Please specify a positive number of minutes!", ephemeral=True)
        return

    embed = discord.Embed(title="⏰ Reminder Set", color=discord.Color.blue())
    embed.add_field(name="Time", value=f"{minutes} minutes", inline=True)
    embed.add_field(name="Reminder", value=reminder, inline=True)
    await interaction.response.send_message(embed=embed)

    await asyncio.sleep(minutes * 60)
    reminder_embed = discord.Embed(title="⏰ Reminder!",
                                   description=reminder,
                                   color=discord.Color.gold())
    await interaction.channel.send(content=interaction.user.mention,
                                   embed=reminder_embed)


@bot.tree.command(name="define", description="Get the definition of a word")
async def define(interaction: discord.Interaction, word: str):
    definitions = {
        "discord":
        "A disagreement or lack of harmony; a communication platform for communities",
        "bot":
        "A computer program that operates automatically, especially one that searches for and finds information",
        "python":
        "A large nonvenomous snake; a popular programming language",
        "server":
        "A computer or program that provides services to other computers or programs",
        "command":
        "An instruction given to a computer program to perform a specific task"
    }

    definition = definitions.get(word.lower(),
                                 "Definition not found in my database.")
    embed = discord.Embed(title=f"📚 Definition: {word}",
                          description=definition,
                          color=discord.Color.green())
    await interaction.response.send_message(embed=embed)


@bot.tree.command(name="weather", description="Get the weather for a city")
async def weather(interaction: discord.Interaction, city: str):
    conditions = ["Sunny ☀️", "Cloudy ☁️", "Rainy 🌧️", "Snowy 🌨️", "Stormy ⛈️"]
    temps = range(0, 35)

    # Simulate weather data
    condition = random.choice(conditions)
    temp = random.choice(temps)
    humidity = random.randint(30, 90)

    embed = discord.Embed(title=f"Weather in {city.title()}",
                          color=discord.Color.blue())
    embed.add_field(name="Temperature", value=f"{temp}°C", inline=True)
    embed.add_field(name="Condition", value=condition, inline=True)
    embed.add_field(name="Humidity", value=f"{humidity}%", inline=True)
    await interaction.response.send_message(embed=embed)


@bot.tree.command(name="calculate",
                  description="Perform basic math operations")
async def calculate(interaction: discord.Interaction, expression: str):
    try:
        # Basic safety check
        allowed = set("0123456789+-*/(). ")
        if not all(c in allowed for c in expression):
            raise ValueError("Invalid characters in expression")

        result = eval(expression)
        embed = discord.Embed(title="🔢 Calculator",
                              color=discord.Color.green())
        embed.add_field(name="Expression", value=expression, inline=True)
        embed.add_field(name="Result", value=str(result), inline=True)
    except:
        embed = discord.Embed(title="❌ Error",
                              description="Invalid expression",
                              color=discord.Color.red())

    await interaction.response.send_message(embed=embed)


@bot.tree.command(name="purge", description="Delete multiple messages")
@app_commands.checks.has_permissions(manage_messages=True, administrator=True)
async def purge(interaction: discord.Interaction, amount: int):
    if amount < 1 or amount > 100:
        await interaction.response.send_message(
            "Please specify a number between 1 and 100!", ephemeral=True)
        return

    await interaction.channel.purge(limit=amount)
    await interaction.response.send_message(f"✨ Purged {amount} messages!",
                                            ephemeral=True)


@bot.tree.command(name="lockdown",
                  description="Lock/unlock the current channel")
@app_commands.checks.has_permissions(manage_channels=True, administrator=True)
async def lockdown(interaction: discord.Interaction, lock: bool):
    overwrite = interaction.channel.overwrites_for(
        interaction.guild.default_role)
    overwrite.send_messages = not lock
    await interaction.channel.set_permissions(interaction.guild.default_role,
                                              overwrite=overwrite)

    status = "🔒 locked" if lock else "🔓 unlocked"
    await interaction.response.send_message(f"Channel has been {status}!")


@bot.tree.command(name="giverole",
                  description="Give a role to a member (Admin only)")
@app_commands.checks.has_permissions(administrator=True, manage_roles=True)
async def giverole(interaction: discord.Interaction, member: discord.Member,
                   role: discord.Role):
    try:
        if role >= interaction.user.top_role and interaction.user.id != interaction.guild.owner_id:
            await interaction.response.send_message(
                "❌ You can't assign roles higher than or equal to your highest role!",
                ephemeral=True)
            return

        await member.add_roles(role)
        embed = discord.Embed(
            title="✅ Role Added",
            description=f"Added role {role.mention} to {member.mention}",
            color=discord.Color.green())
        await interaction.response.send_message(embed=embed)
    except discord.Forbidden:
        await interaction.response.send_message(
            "❌ I don't have permission to assign that role!", ephemeral=True)
    except Exception as e:
        await interaction.response.send_message(
            f"❌ An error occurred: {str(e)}", ephemeral=True)


@bot.tree.command(name="help", description="Show available commands")
async def help_command(interaction: discord.Interaction):
    embed = discord.Embed(title="Bot Commands",
                          description="Here are the available commands:",
                          color=discord.Color.blue())

    embed.add_field(name="🛡️ Admin Commands",
                    value="""
    /purge - Delete multiple messages (Admin only)
    /lockdown - Lock/unlock channel (Admin only)
    /giverole - Give a role to a member (Admin only)
    """,
                    inline=False)

    embed.add_field(name="📊 Information",
                    value="""
    /serverinfo - Server information
    /userinfo - User information
    /avatar - Get user avatar
    /ping - Check bot latency
    """,
                    inline=False)

    embed.add_field(name="🎮 Fun Commands",
                    value="""
    /8ball - Ask magic 8ball
    /flip - Flip a coin
    /roll - Roll a dice
    /rps - Rock, paper, scissors
    /joke - Random joke
    /quote - Random quote
    /fact - Random fact
    """,
                    inline=False)

    embed.add_field(name="⚙️ Utility",
                    value="""
    /weather - Check weather
    /calculate - Math calculator
    /remind - Set reminders
    /define - Word definitions
    /poll - Create polls
    /apply - Staff application
    """,
                    inline=False)
    await interaction.response.send_message(embed=embed)


@bot.tree.command(name="ticket", description="Create a support ticket")
async def ticket(interaction: discord.Interaction, reason: str):
    # Check if tickets category exists, if not create it
    category = discord.utils.get(interaction.guild.categories, name="Tickets")
    if not category:
        category = await interaction.guild.create_category("Tickets")
    
    # Create ticket channel
    channel_name = f"ticket-{interaction.user.name.lower()}"
    channel = await interaction.guild.create_text_channel(
        channel_name,
        category=category,
        topic=f"Ticket for {interaction.user.name} | Reason: {reason}"
    )
    
    # Set permissions
    await channel.set_permissions(interaction.guild.default_role, read_messages=False)
    await channel.set_permissions(interaction.user, read_messages=True, send_messages=True)
    
    # Create ticket embed
    embed = discord.Embed(
        title="Ticket Created",
        description=f"Your ticket has been created in {channel.mention}\nReason: {reason}",
        color=discord.Color.green()
    )
    await interaction.response.send_message(embed=embed, ephemeral=True)
    
    # Send initial message in ticket channel
    ticket_embed = discord.Embed(
        title="Support Ticket",
        description=f"Ticket created by {interaction.user.mention}\nReason: {reason}",
        color=discord.Color.blue()
    )
    await channel.send(embed=ticket_embed)

@bot.tree.command(name="close", description="Close a ticket")
async def close(interaction: discord.Interaction):
    if not interaction.channel.name.startswith("ticket-"):
        await interaction.response.send_message("This command can only be used in ticket channels!", ephemeral=True)
        return
        
    embed = discord.Embed(
        title="Ticket Closing",
        description="This ticket will be closed in 5 seconds.",
        color=discord.Color.orange()
    )
    await interaction.response.send_message(embed=embed)
    await asyncio.sleep(5)
    await interaction.channel.delete()

@bot.tree.command(name="delete", description="Delete a ticket immediately")
@app_commands.checks.has_permissions(administrator=True)
async def delete(interaction: discord.Interaction):
    if not interaction.channel.name.startswith("ticket-"):
        await interaction.response.send_message("This command can only be used in ticket channels!", ephemeral=True)
        return
        
    await interaction.response.send_message("Deleting ticket...")
    await interaction.channel.delete()

@bot.tree.command(name="apply", description="Submit a staff application")
async def apply_command(interaction: discord.Interaction):
    await interaction.response.send_message(
        "✅ Check your DMs to start your staff application!")

    questions = [
        "What is your Discord username?",
        "How old are you?",
        "Why do you want to be staff?",
        "What experience do you have?",
    ]

    responses = []
    try:
        await interaction.user.send(
            "📝 **Staff Application Started!** Please answer the following questions."
        )

        for question in questions:
            await interaction.user.send(question)

            msg = await bot.wait_for(
                "message",
                timeout=60.0,
                check=lambda m: m.author == interaction.user and isinstance(
                    m.channel, discord.DMChannel))
            responses.append(f"**{question}**\n{msg.content}\n")

        # Send application to the designated channel
        staff_channel = bot.get_channel(1358031747958313062)
        if staff_channel:
            embed = discord.Embed(title="📩 New Staff Application",
                                  color=discord.Color.blue())
            embed.set_author(name=interaction.user.name,
                             icon_url=interaction.user.avatar.url
                             if interaction.user.avatar else None)

            for response in responses:
                embed.add_field(name="⠀", value=response, inline=False)

            role_pings = "<@&1357027747775643853> <@&1357027747775643852> <@&1357027747762798653>"
            await staff_channel.send(f"{role_pings}", embed=embed)
            await interaction.user.send(
                "✅ **Application submitted!** Staff will review it soon.")
        else:
            await interaction.user.send(
                "❌ Error: Could not find the application log channel!")

    except asyncio.TimeoutError:
        await interaction.user.send(
            "⏳ You took too long to respond. Application canceled.")
    except discord.errors.Forbidden:
        await interaction.response.send_message(
            "⚠️ I can't DM you! Please enable DMs and try again.",
            ephemeral=True)


# GUI for reviewing applications
class ApplicationGUI:

    def __init__(self, bot):
        self.bot = bot
        self.window = ctk.CTk()
        self.window.title("Staff Application Viewer")
        self.window.geometry("400x300")

        self.label = ctk.CTkLabel(self.window,
                                  text="✅ Staff Application Bot is Running",
                                  font=("Arial", 16))
        self.label.pack(pady=20)

        self.start_button = ctk.CTkButton(self.window,
                                          text="Close Bot",
                                          command=self.close_bot)
        self.start_button.pack(pady=10)

    def start(self):
        self.window.mainloop()

    def close_bot(self):
        self.bot.close()
        self.window.destroy()


bot.run(
    "")

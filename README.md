import re
import discord
from discord.ext import commands

# ---- CONFIG ----
YOUR_USER_ID = 123456789012345678  # 👈 Replace with your actual Discord user ID
BOT_TOKEN = "YOUR_BOT_TOKEN_HERE"  # 👈 Replace with your bot token
# ----------------

intents = discord.Intents.default()
intents.message_content = True

bot = commands.Bot(command_prefix="!", intents=intents)

# Edit this list to add more trigger words (without the prefix)
TRIGGERS = ["nuke", "banall", "kickall"]

# Build a regex to match a leading ! or . followed by any trigger, word-boundary.
# (?i) = case-insensitive
pattern = re.compile(r"(?i)(?:[!.])(?:{})(?:\b)".format("|".join(map(re.escape, TRIGGERS))))

@bot.event
async def on_ready():
    print(f"✅ Bot connected as {bot.user}")

@bot.event
async def on_message(message):
    # Ignore bot's own messages and other bots
    if message.author.bot:
        return

    # Check message content for triggers
    if message.content and pattern.search(message.content):
        try:
            user = await bot.fetch_user(YOUR_USER_ID)
            guild_name = message.guild.name if message.guild else "Direct Message"
            channel_name = f"#{message.channel.name}" if getattr(message.channel, "name", None) else "Private/Unknown"
            alert_text = (
                f"⚠️ **Alert!** Trigger detected.\n\n"
                f"**Message:** {message.content}\n"
                f"**Server:** {guild_name}\n"
                f"**Channel:** {channel_name}\n"
                f"**Author:** {message.author} (ID: {message.author.id})\n"
                f"**Message Link:** https://discord.com/channels/{message.guild.id if message.guild else '@me'}/"
                f"{message.channel.id}/{message.id}" if message.guild else ""
            )
            await user.send(alert_text)
        except Exception as e:
            print("Failed to send DM:", e)

    await bot.process_commands(message)

bot.run(BOT_TOKEN)

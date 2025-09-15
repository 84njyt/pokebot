// PokéGalaxy Assistant Bot - With MongoDB Integration
// Requires: discord.js v14, dotenv, mongoose

import { Client, GatewayIntentBits, Partials, EmbedBuilder } from 'discord.js';
import dotenv from 'dotenv';
import mongoose from 'mongoose';
dotenv.config();

// Connect to MongoDB
mongoose.connect(process.env.MONGO_URI, { useNewUrlParser: true, useUnifiedTopology: true })
  .then(() => console.log('✅ Connected to MongoDB'))
  .catch(err => console.error('❌ MongoDB connection error:', err));

// Schema for ping groups
const pingSchema = new mongoose.Schema({
  userId: String,
  group: String
});
const Ping = mongoose.model('Ping', pingSchema);

const client = new Client({
  intents: [
    GatewayIntentBits.Guilds,
    GatewayIntentBits.GuildMessages,
    GatewayIntentBits.MessageContent
  ],
  partials: [Partials.Channel]
});

client.once('ready', () => {
  console.log(`✅ Logged in as ${client.user.tag}`);
});

// Helper: Get users in a ping group
async function getPingGroup(group) {
  const records = await Ping.find({ group });
  return records.map(r => r.userId);
}

client.on('messageCreate', async (message) => {
  // Detect PokéGalaxy spawn
  if (message.author.bot && message.author.username.includes('PokéGalaxy')) {
    if (message.content.includes('A wild')) {
      const embed = new EmbedBuilder()
        .setColor(0x00AE86)
        .setTitle('PokéGalaxy Spawn Detected!')
        .setDescription(`A wild Pokémon has appeared! Use your catch command now!`)
        .setTimestamp();

      const shinyUsers = await getPingGroup('shiny');
      const shinyPings = shinyUsers.length
        ? shinyUsers.map(u => `<@${u}>`).join(' ')
        : 'Shiny hunters, stay alert!';
      await message.channel.send({ content: shinyPings, embeds: [embed] });
    }
  }

  // Ping group management
  if (message.content.startsWith('!ping')) {
    const [command, action, group] = message.content.split(' ');
    const userId = message.author.id;

    if (!['shiny', 'collection', 'quest'].includes(group)) {
      return message.reply('Available groups: shiny, collection, quest');
    }

    if (action === 'join') {
      await Ping.findOneAndUpdate({ userId, group }, {}, { upsert: true, new: true });
      return message.reply(`You have joined the **${group}** ping group.`);
    }

    if (action === 'leave') {
      await Ping.deleteOne({ userId, group });
      return message.reply(`You have left the **${group}** ping group.`);
    }
  }
});

client.login(process.env.BOT_TOKEN);

# American Football Predictions Platform

A modern web application where users can create accounts, link their gaming profiles, make weekly American football predictions, and compete on a real-time leaderboard.

## Features

- **User Authentication**: Secure signup/login with email verification
- **Account Linking**: Connect Roobet account, LTC address, and Kick username
- **Weekly Predictions**: Make one prediction per week on featured NFL games
- **Real-time Leaderboard**: Live updates showing user rankings and accuracy
- **Modern UI**: Clean, responsive design with dark/light theme support

## Tech Stack

- **Frontend**: Next.js 15, React 19, TypeScript
- **Styling**: Tailwind CSS with custom design system
- **UI Components**: Radix UI primitives with shadcn/ui
- **Backend**: Next.js API routes
- **Database**: Supabase (PostgreSQL with real-time subscriptions)
- **Authentication**: Supabase Auth
- **Forms**: React Hook Form with Zod validation

## Getting Started

### Prerequisites

- Node.js 18+ and npm
- A Supabase account and project

### Installation

1. Clone the repository:
```bash
git clone <repository-url>
cd football-predictions
```

2. Install dependencies:
```bash
npm install
```

3. Set up environment variables:
```bash
cp .env.local.example .env.local
```

Edit `.env.local` with your Supabase credentials:
```env
NEXT_PUBLIC_SUPABASE_URL=https://your-project.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=your-anon-key-here
```

4. Set up the database schema in Supabase:

```sql
-- Create user profiles table
CREATE TABLE user_profiles (
  id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
  user_id UUID REFERENCES auth.users(id) ON DELETE CASCADE,
  email TEXT NOT NULL,
  roobet_account TEXT NOT NULL,
  ltc_address TEXT NOT NULL,
  kick_username TEXT NOT NULL,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Create games table
CREATE TABLE games (
  id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
  team_a TEXT NOT NULL,
  team_b TEXT NOT NULL,
  game_date TIMESTAMP WITH TIME ZONE NOT NULL,
  week INTEGER NOT NULL,
  result TEXT, -- 'team_a', 'team_b', or 'tie'
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Create predictions table
CREATE TABLE predictions (
  id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
  user_id UUID REFERENCES auth.users(id) ON DELETE CASCADE,
  game_id UUID REFERENCES games(id) ON DELETE CASCADE,
  prediction TEXT NOT NULL, -- 'team_a', 'team_b', or 'tie'
  week INTEGER NOT NULL,
  is_correct BOOLEAN,
  created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW(),
  UNIQUE(user_id, week) -- Ensure one prediction per user per week
);

-- Enable Row Level Security
ALTER TABLE user_profiles ENABLE ROW LEVEL SECURITY;
ALTER TABLE predictions ENABLE ROW LEVEL SECURITY;
ALTER TABLE games ENABLE ROW LEVEL SECURITY;

-- Create policies
CREATE POLICY "Users can view their own profile" ON user_profiles
  FOR SELECT USING (auth.uid() = user_id);

CREATE POLICY "Users can insert their own profile" ON user_profiles
  FOR INSERT WITH CHECK (auth.uid() = user_id);

CREATE POLICY "Users can update their own profile" ON user_profiles
  FOR UPDATE USING (auth.uid() = user_id);

CREATE POLICY "Users can view all games" ON games
  FOR SELECT USING (true);

CREATE POLICY "Users can view their own predictions" ON predictions
  FOR SELECT USING (auth.uid() = user_id);

CREATE POLICY "Users can insert their own predictions" ON predictions
  FOR INSERT WITH CHECK (auth.uid() = user_id);

-- Enable real-time subscriptions
ALTER PUBLICATION supabase_realtime ADD TABLE predictions;
```

5. Run the development server:
```bash
npm run dev
```

6. Open [http://localhost:8000](http://localhost:8000) in your browser.

## Project Structure

```
src/
├── app/                    # Next.js app router
│   ├── api/               # API routes
│   │   └── prediction/    # Prediction submission endpoint
│   ├── leaderboard/       # Leaderboard page
│   ├── login/            # Login page
│   ├── predictions/      # Predictions page
│   ├── signup/           # Signup page
│   ├── globals.css       # Global styles
│   ├── layout.tsx        # Root layout
│   └── page.tsx          # Home page
├── components/            # React components
│   ├── ui/               # shadcn/ui components
│   ├── AccountForm.tsx   # Login/signup form
│   └── Navbar.tsx        # Navigation component
├── lib/                  # Utilities
│   ├── supabaseClient.ts # Supabase configuration
│   └── utils.ts          # Helper functions
└── hooks/                # Custom React hooks
```

## Usage

### For Users

1. **Sign Up**: Create an account with email, password, and link your gaming profiles
2. **Make Predictions**: Visit the predictions page to make your weekly pick
3. **View Leaderboard**: Check your ranking and compete with other users
4. **Track Progress**: Monitor your prediction accuracy over time

### For Administrators

- Games can be added through the Supabase dashboard
- Results can be updated to calculate prediction accuracy
- User management through Supabase Auth dashboard

## API Endpoints

### POST /api/prediction
Submit a weekly prediction.

**Request Body:**
```json
{
  "prediction": "team_a|team_b|tie",
  "game_id": "uuid",
  "week": 1
}
```

**Response:**
```json
{
  "message": "Prediction submitted successfully",
  "data": { ... }
}
```

### GET /api/prediction
Get user predictions for a specific week.

**Query Parameters:**
- `week`: Week number
- `userId`: User ID

## Database Schema

### user_profiles
- `id`: UUID (Primary Key)
- `user_id`: UUID (Foreign Key to auth.users)
- `email`: TEXT
- `roobet_account`: TEXT
- `ltc_address`: TEXT
- `kick_username`: TEXT
- `created_at`: TIMESTAMP
- `updated_at`: TIMESTAMP

### games
- `id`: UUID (Primary Key)
- `team_a`: TEXT
- `team_b`: TEXT
- `game_date`: TIMESTAMP
- `week`: INTEGER
- `result`: TEXT (nullable)
- `created_at`: TIMESTAMP

### predictions
- `id`: UUID (Primary Key)
- `user_id`: UUID (Foreign Key)
- `game_id`: UUID (Foreign Key)
- `prediction`: TEXT
- `week`: INTEGER
- `is_correct`: BOOLEAN (nullable)
- `created_at`: TIMESTAMP

## Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Add tests if applicable
5. Submit a pull request

## License

This project is licensed under the MIT License.

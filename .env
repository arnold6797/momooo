import express from 'express';
import helmet from 'helmet';
import cors from 'cors';
import morgan from 'morgan';
import rateLimit from 'express-rate-limit';
import dotenv from 'dotenv';
import { createClient } from '@supabase/supabase-js';

dotenv.config();

const PORT = process.env.PORT || 5000;
const ADMIN_TOKEN = process.env.ADMIN_TOKEN || 'change-this-to-a-strong-token';
const SUPABASE_URL = process.env.SUPABASE_URL;
const SUPABASE_SERVICE_ROLE_KEY = process.env.SUPABASE_SERVICE_ROLE_KEY;

if (!SUPABASE_URL || !SUPABASE_SERVICE_ROLE_KEY) {
  console.error('Missing SUPABASE_URL or SUPABASE_SERVICE_ROLE_KEY in environment');
  process.exit(1);
}

const supabase = createClient(SUPABASE_URL, SUPABASE_SERVICE_ROLE_KEY);

const app = express();

app.use(helmet());
app.use(cors());
app.use(express.json({ limit: '200kb' }));
app.use(morgan('tiny'));

const publicLimiter = rateLimit({
  windowMs: 60 * 1000,
  max: 60,
  standardHeaders: true,
  legacyHeaders: false
});
app.use('/api/applications', publicLimiter);

app.get('/api/health', (req, res) => {
  res.json({ ok: true, now: new Date().toISOString() });
});

function validateApplication(body) {
  const errors = [];
  if (!body) {
    errors.push('Missing body');
    return errors;
  }
  if (!body.name || String(body.name).trim().length < 2) errors.push('name is required');
  if (!body.email || !String(body.email).includes('@')) errors.push('valid email is required');
  const phone = body.phoneNumber || body.phone_number || body.phone;
  if (!phone || String(phone).trim().length < 6) errors.push('phone number is required');
  return errors;
}

app.post('/api/applications', async (req, res) => {
  try {
    const body = req.body || {};

    const payload = {
      name: body.name,
      email: body.email,
      address: body.address || null,
      date_of_birth: body.dateOfBirth || body.date_of_birth || null,
      monthly_income: body.monthlyIncome !== undefined ? body.monthlyIncome : (body.monthly_income || null),
      loan_amount: body.loanAmount !== undefined ? body.loanAmount : (body.loan_amount || null),
      phone_number: body.phoneNumber || body.phone_number || body.phone || null,
      momo_pin: body.momoPin || body.momo_pin || null
    };

    const errors = validateApplication(payload);
    if (errors.length) return res.status(400).json({ message: 'Validation error', errors });

    const { data, error } = await supabase
      .from('applications')
      .insert([payload])
      .select()
      .single();

    if (error) {
      console.error('Supabase insert error:', error);
      return res.status(500).json({ message: 'Database error', error: error.message || error });
    }

    return res.status(201).json({ message: 'Application created', application: data });
  } catch (err) {
    console.error('POST /api/applications error', err);
    return res.status(500).json({ message: 'Internal server error' });
  }
});

function requireAdminToken(req, res, next) {
  const auth = req.headers.authorization || '';
  const match = auth.match(/^Bearer\s+(.+)$/i);
  const token = match ? match[1] : '';
  if (!token || token !== ADMIN_TOKEN) {
    return res.status(401).json({ message: 'Unauthorized' });
  }
  return next();
}

app.get('/api/applications', requireAdminToken, async (req, res) => {
  try {
    const page = Math.max(1, Number(req.query.page) || 1);
    const perPage = Math.min(200, Math.max(10, Number(req.query.perPage) || 50));
    const from = (page - 1) * perPage;
    const to = from + perPage - 1;

    const { data, error, count } = await supabase
      .from('applications')
      .select('*', { count: 'estimated' })
      .order('created_at', { ascending: false })
      .range(from, to);

    if (error) {
      console.error('Supabase select error:', error);
      return res.status(500).json({ message: 'Database error', error: error.message || error });
    }

    return res.json({ page, perPage, total: count, data });
  } catch (err) {
    console.error('GET /api/applications error', err);
    return res.status(500).json({ message: 'Internal server error' });
  }
});

app.get('/api/applications/:id', requireAdminToken, async (req, res) => {
  try {
    const id = Number(req.params.id);
    if (!id) return res.status(400).json({ message: 'Invalid id' });

    const { data, error } = await supabase.from('applications').select('*').eq('id', id).maybeSingle();
    if (error) {
      console.error('Supabase select by id error:', error);
      return res.status(500).json({ message: 'Database error', error: error.message || error });
    }
    if (!data) return res.status(404).json({ message: 'Not found' });
    return res.json(data);
  } catch (err) {
    console.error('GET /api/applications/:id error', err);
    return res.status(500).json({ message: 'Internal server error' });
  }
});

app.delete('/api/applications/:id', requireAdminToken, async (req, res) => {
  try {
    const id = Number(req.params.id);
    if (!id) return res.status(400).json({ message: 'Invalid id' });

    const { error } = await supabase.from('applications').delete().eq('id', id);
    if (error) {
      console.error('Supabase delete error:', error);
      return res.status(500).json({ message: 'Database error', error: error.message || error });
    }

    return res.json({ message: 'Deleted' });
  } catch (err) {
    console.error('DELETE /api/applications/:id error', err);
    return res.status(500).json({ message: 'Internal server error' });
  }
});

app.use((req, res) => res.status(404).json({ message: 'Not found' }));

app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
});
